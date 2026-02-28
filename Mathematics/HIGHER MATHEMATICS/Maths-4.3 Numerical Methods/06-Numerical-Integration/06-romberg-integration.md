# Chapter 6.6: Romberg Integration

[← Previous: Weddle's Rule](05-weddles-rule.md) | [Next: Gaussian Quadrature →](07-gaussian-quadrature.md)

---

## Overview

**Romberg integration** systematically combines trapezoidal rule estimates with successively halved step sizes using **Richardson extrapolation** to achieve high-order accuracy without deriving new formulas. It's an elegant "meta-method" that bootstraps the simple trapezoidal rule into a powerhouse.

---

## 1. The Key Idea

The trapezoidal rule with step $h$ has error expansion:

$$T(h) = I + c_1 h^2 + c_2 h^4 + c_3 h^6 + \cdots$$

By computing $T(h)$ and $T(h/2)$, we can **eliminate** the $h^2$ error:

$$\frac{4T(h/2) - T(h)}{3} = I + O(h^4)$$

This is exactly **Simpson's rule**! Continuing:

$$\frac{16S(h/2) - S(h)}{15} = I + O(h^6) \quad \text{(Boole!)}$$

---

## 2. The Romberg Table

$$R(n, m) = \frac{4^m R(n, m-1) - R(n-1, m-1)}{4^m - 1}$$

```
  R(0,0) = T(h)
  R(1,0) = T(h/2)     → R(1,1) = Simpson
  R(2,0) = T(h/4)     → R(2,1)       → R(2,2) = Boole-like
  R(3,0) = T(h/8)     → R(3,1)       → R(3,2)       → R(3,3)
  ↓                      ↓               ↓               ↓
  O(h²)               O(h⁴)          O(h⁶)          O(h⁸)
```

---

## 3. Algorithm

1. Compute $R(0,0) = \frac{h}{2}[f(a) + f(b)]$ with $h = b-a$
2. For $n = 1, 2, 3, \ldots$:
   - Halve $h$: $h_n = h/2^n$
   - Compute $R(n,0)$ using the composite trapezoidal rule
   - For $m = 1, 2, \ldots, n$:
     $$R(n,m) = \frac{4^m R(n,m-1) - R(n-1,m-1)}{4^m - 1}$$
3. Stop when $|R(n,n) - R(n-1,n-1)| < \text{tolerance}$

---

## 4. Worked Example

Evaluate $\int_0^1 e^x\,dx$ using Romberg integration.

### Step 1: Trapezoidal estimates

$R(0,0) = \frac{1}{2}[e^0 + e^1] = \frac{1+2.71828}{2} = 1.85914$

$R(1,0) = \frac{0.5}{2}[1 + 2(e^{0.5}) + e^1] = 0.25[1 + 3.29744 + 2.71828] = 1.75393$

$R(2,0) = \frac{0.25}{2}[1 + 2(e^{0.25} + e^{0.5} + e^{0.75}) + e^1]$

$= 0.125[1 + 2(1.28403 + 1.64872 + 2.11700) + 2.71828]$

$= 0.125[1 + 10.09950 + 2.71828] = 1.72722$

### Step 2: Build the table

| $n$ | $R(n,0)$ | $R(n,1)$ | $R(n,2)$ |
|-----|----------|----------|----------|
| 0 | 1.85914 | | |
| 1 | 1.75393 | **1.71886** | |
| 2 | 1.72722 | **1.71833** | **1.71829** |

**Extrapolation formulas:**

$R(1,1) = \frac{4(1.75393) - 1.85914}{3} = \frac{7.01572 - 1.85914}{3} = \frac{5.15658}{3} = 1.71886$

$R(2,1) = \frac{4(1.72722) - 1.75393}{3} = \frac{6.90888 - 1.75393}{3} = \frac{5.15495}{3} = 1.71832$

$R(2,2) = \frac{16(1.71832) - 1.71886}{15} = \frac{27.49312 - 1.71886}{15} = \frac{25.77426}{15} = 1.71828$

**Exact:** $e - 1 = 1.71828$ ✓ (accurate to 5 decimal places with only 5 function evaluations!)

---

## 5. Advantages

```
  ┌─────────────────────────────────────────────┐
  │         Why Romberg is Powerful              │
  ├─────────────────────────────────────────────┤
  │ • Reuses ALL previous function evaluations  │
  │ • Automatically increases accuracy order    │
  │ • Built-in error estimate: R(n,n) - R(n-1,n-1)│
  │ • Only needs trapezoidal code as base       │
  │ • Achieves O(h^{2n+2}) at column n         │
  └─────────────────────────────────────────────┘
```

---

## 6. Connection to Other Methods

| Romberg Column | Equivalent to |
|----------------|--------------|
| $R(n,0)$ | Composite Trapezoidal, $O(h^2)$ |
| $R(n,1)$ | Composite Simpson, $O(h^4)$ |
| $R(n,2)$ | Boole-type, $O(h^6)$ |
| $R(n,3)$ | $O(h^8)$ |
| $R(n,m)$ | $O(h^{2m+2})$ |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Method** | Richardson extrapolation of trapezoidal rule |
| **Recurrence** | $R(n,m) = \frac{4^m R(n,m-1) - R(n-1,m-1)}{4^m-1}$ |
| **Column $m$ accuracy** | $O(h^{2m+2})$ |
| **Error estimate** | $|R(n,n) - R(n-1,n-1)|$ |
| **Function evals** | All reused; very efficient |
| **Limitation** | Requires equally spaced data; step must halve |

---

## Quick Revision Questions

1. **Write the Romberg extrapolation formula.**

2. **Show that $R(1,1)$ from two trapezoidal estimates gives Simpson's rule.**

3. **Build a Romberg table for $\int_0^2 x^2\,dx$ with $R(0,0)$, $R(1,0)$, and $R(1,1)$.**

4. **What is the order of accuracy of $R(n,2)$?**

5. **Why is Romberg integration more efficient than simply refining the trapezoidal rule?**

6. **How is the stopping criterion implemented in Romberg integration?**

---

[← Previous: Weddle's Rule](05-weddles-rule.md) | [Next: Gaussian Quadrature →](07-gaussian-quadrature.md)
