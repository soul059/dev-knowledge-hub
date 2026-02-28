# Chapter 3: Matrix Exponentiation for DP

## 📋 Overview
When a DP recurrence is **linear** and has **constant coefficients**, the transition can be expressed as **matrix multiplication**. Using **fast matrix exponentiation** ($O(\log n)$ multiplications), we solve in $O(k^3 \log n)$ instead of $O(kn)$, where $k$ is the state size.

---

## 🧠 Core Idea

### Linear Recurrence as Matrix
```
Fibonacci: F(n) = F(n-1) + F(n-2)

Matrix form:
┌         ┐   ┌       ┐   ┌         ┐
│ F(n)    │ = │ 1   1 │ · │ F(n-1)  │
│ F(n-1)  │   │ 1   0 │   │ F(n-2)  │
└         ┘   └       ┘   └         ┘

Therefore:
┌         ┐       ┌       ┐ⁿ⁻¹   ┌      ┐
│ F(n)    │   =   │ 1   1 │     · │ F(1) │
│ F(n-1)  │       │ 1   0 │       │ F(0) │
└         ┘       └       ┘       └      ┘

Compute M^(n-1) using fast exponentiation!
```

### Fast Matrix Exponentiation
```
M^n using divide and conquer:

function matpow(M, n):
    if n == 1: return M
    if n is even:
        half = matpow(M, n/2)
        return half * half
    else:
        return M * matpow(M, n-1)

Multiplications: O(log n)
Each multiplication: O(k³)  for k×k matrix
Total: O(k³ · log n)
```

---

## 🔍 Step-by-Step Trace: Fibonacci

```
F(10) using matrix exponentiation:

M = [[1,1],[1,0]]

M^9 = M^8 · M^1
M^8 = (M^4)²
M^4 = (M^2)²
M^2 = M · M

M^1 = [[1,1],[1,0]]

M^2 = [[1,1],[1,0]] · [[1,1],[1,0]]
    = [[2,1],[1,1]]

M^4 = [[2,1],[1,1]] · [[2,1],[1,1]]
    = [[5,3],[3,2]]

M^8 = [[5,3],[3,2]] · [[5,3],[3,2]]
    = [[34,21],[21,13]]

M^9 = [[34,21],[21,13]] · [[1,1],[1,0]]
    = [[55,34],[34,21]]

Result: M^9 · [1, 0]ᵀ = [55, 34]ᵀ
F(10) = 55 ✓
```

---

## 📊 Common Recurrences as Matrices

### Tribonacci
```
T(n) = T(n-1) + T(n-2) + T(n-3)

┌        ┐   ┌           ┐   ┌        ┐
│ T(n)   │   │ 1   1   1 │   │ T(n-1) │
│ T(n-1) │ = │ 1   0   0 │ · │ T(n-2) │
│ T(n-2) │   │ 0   1   0 │   │ T(n-3) │
└        ┘   └           ┘   └        ┘

Matrix size: 3×3
Complexity: O(3³ · log n) = O(27 · log n)
```

### Climbing Stairs (1 or 2 steps)
```
Same as Fibonacci!
dp[n] = dp[n-1] + dp[n-2]

M = [[1,1],[1,0]], exponent = n
```

### Climbing Stairs (1, 2, or 3 steps)
```
Same as Tribonacci!
dp[n] = dp[n-1] + dp[n-2] + dp[n-3]

M = [[1,1,1],[1,0,0],[0,1,0]], exponent = n
```

### General Linear Recurrence
```
dp[n] = c₁·dp[n-1] + c₂·dp[n-2] + ... + cₖ·dp[n-k]

Companion matrix (k×k):
┌                          ┐
│ c₁  c₂  c₃  ...  cₖ₋₁ cₖ│
│  1   0   0  ...   0    0 │
│  0   1   0  ...   0    0 │
│  .   .   .  ...   .    . │
│  0   0   0  ...   1    0 │
└                          ┘
```

---

## 💻 Pseudocode

```
function matMul(A, B, mod):
    // A is p×q, B is q×r → result is p×r
    C = new matrix [p][r] = 0
    for i = 0 to p-1:
        for j = 0 to r-1:
            for k = 0 to q-1:
                C[i][j] = (C[i][j] + A[i][k] * B[k][j]) % mod
    return C

function matPow(M, n, mod):
    k = size of M
    result = identity matrix (k×k)
    base = M
    
    while n > 0:
        if n & 1:
            result = matMul(result, base, mod)
        base = matMul(base, base, mod)
        n >>= 1
    
    return result

function fibonacci(n, mod):
    if n <= 1: return n
    M = [[1, 1], [1, 0]]
    Mn = matPow(M, n-1, mod)
    return Mn[0][0]  // F(n) = Mn[0][0] * F(1) + Mn[0][1] * F(0)
```

---

## 🌐 Advanced Application: Tiling Problems

### 2×n Tiling with 1×2 Dominoes
```
dp[n] = dp[n-1] + dp[n-2]
  Place vertical domino: n-1 remaining
  Place two horizontal: n-2 remaining

Same as Fibonacci → use matrix exponentiation for huge n
```

### 3×n Tiling
```
More complex: profile DP combined with matrix exponentiation

State: bitmask of partially filled cells in current column
Transition matrix: which profiles can follow which

Build k×k matrix (k = 2³ = 8 for width 3)
Raise to power n → answer

Time: O(k³ · log n) where k = 2^width
```

---

## 🧩 When to Use Matrix Exponentiation

```
✓ Linear recurrence with constant coefficients
✓ Very large n (10⁹ or larger)
✓ Need answer modulo some number
✓ State space is small (k ≤ ~100)

✗ Non-linear recurrence (dp[n] = dp[n-1]²)
✗ Variable coefficients (depend on n)
✗ State space too large (matrix multiply too expensive)
✗ Small n (regular DP is simpler and fast enough)

Sweet spot: small k, huge n
  k=2, n=10¹⁸: O(8 · 60) = 480 operations ✓
  k=100, n=10⁶: O(10⁶ · 20) = 2·10⁷ vs O(10⁸) ← marginal
```

---

## ⚡ Complexity Analysis

```
┌──────────────────────────────────────────┐
│ Regular DP:  O(k · n)                    │
│   k = state size, n = steps              │
│                                          │
│ Matrix Exponentiation:  O(k³ · log n)    │
│   k³ per matrix multiplication           │
│   log n multiplications                  │
│                                          │
│ Wins when: n >> k²                       │
│   Fibonacci: k=2, n=10⁹                 │
│   O(2·10⁹) → O(8·30) = O(240) 🚀       │
│                                          │
│ Space: O(k²) for matrices               │
└──────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Idea** | Express linear recurrence as matrix multiplication |
| **Matrix** | Companion matrix encodes coefficients |
| **Fast power** | M^n via repeated squaring: O(log n) |
| **Total time** | O(k³ · log n) for k×k matrix |
| **Use case** | Small state, huge n, constant coefficients |
| **Applications** | Fibonacci, tiling, graph path counting |

---

## ❓ Quick Revision Questions

1. **What form must a recurrence have to use matrix exponentiation?**
2. **Write the companion matrix for F(n) = F(n-1) + F(n-2).**
3. **What is the time complexity of computing M^n for a k×k matrix?**
4. **When does matrix exponentiation outperform regular DP?**
5. **How do you handle modular arithmetic in matrix exponentiation?**
6. **How would you apply this to a 3×n tiling problem?**

---

[← Previous: Rolling Array](02-rolling-array.md) | [Next: Convex Hull Trick →](04-convex-hull-trick.md)

[← Back to README](../README.md)
