# Chapter 4.4: Lagrange Interpolation

[← Previous: Newton's Backward Difference](03-newton-backward.md) | [Next: Divided Differences →](05-divided-differences.md)

---

## Overview

**Lagrange interpolation** constructs the unique polynomial of degree $\leq n$ that passes through $n+1$ given data points. Unlike Newton's formulas, it works for **unequally spaced** data and does NOT require a difference table.

---

## 1. The Formula

Given $n+1$ points $(x_0, f_0), (x_1, f_1), \ldots, (x_n, f_n)$:

$$\boxed{P_n(x) = \sum_{i=0}^{n} f_i \cdot L_i(x)}$$

where the **Lagrange basis polynomials** are:

$$L_i(x) = \prod_{\substack{j=0 \\ j \neq i}}^{n} \frac{x - x_j}{x_i - x_j}$$

### Properties of $L_i(x)$

$$L_i(x_j) = \begin{cases} 1 & \text{if } i = j \\ 0 & \text{if } i \neq j \end{cases} = \delta_{ij}$$

---

## 2. Geometric Interpretation

Each $L_i(x)$ is a polynomial that equals 1 at $x_i$ and 0 at all other data points. The interpolant is a weighted sum:

```
  L₀(x):   1 ●
            │ ╲
            │  ╲
            │   ╲──────────────
  ──────────┼────●────●────●── x
            x₀   x₁   x₂   x₃
            └ = 1 at x₀, = 0 at x₁, x₂, x₃

  P(x) = f₀·L₀(x) + f₁·L₁(x) + f₂·L₂(x) + f₃·L₃(x)
```

---

## 3. Worked Example

Find the polynomial through $(1, 1)$, $(2, 4)$, $(3, 9)$.

$$L_0(x) = \frac{(x-2)(x-3)}{(1-2)(1-3)} = \frac{(x-2)(x-3)}{2}$$

$$L_1(x) = \frac{(x-1)(x-3)}{(2-1)(2-3)} = \frac{(x-1)(x-3)}{-1}$$

$$L_2(x) = \frac{(x-1)(x-2)}{(3-1)(3-2)} = \frac{(x-1)(x-2)}{2}$$

$$P_2(x) = 1 \cdot \frac{(x-2)(x-3)}{2} + 4 \cdot \frac{(x-1)(x-3)}{-1} + 9 \cdot \frac{(x-1)(x-2)}{2}$$

Simplifying: $P_2(x) = x^2$ ✓

---

## 4. Error in Lagrange Interpolation

$$f(x) - P_n(x) = \frac{f^{(n+1)}(\xi)}{(n+1)!} \prod_{i=0}^{n}(x - x_i)$$

for some $\xi$ in the interval containing $x$ and all $x_i$.

---

## 5. Advantages and Disadvantages

| Advantages | Disadvantages |
|-----------|---------------|
| Works for unequal spacing | Adding a new point requires recomputation |
| No difference table needed | Computationally expensive: $O(n^2)$ |
| Elegant and compact formula | Roundoff accumulation for high degree |
| Easy to program | Runge phenomenon for high $n$ |

---

## 6. Runge Phenomenon

For high-degree polynomial interpolation on equally spaced points, oscillations can occur near the edges:

```
  f(x)    True function: f(x) = 1/(1+25x²)
    │
    │  ╱╲                              ╱╲
    │ ╱  ╲     P₁₀(x) oscillates!    ╱  ╲
    │╱    ╲                          ╱    ╲
    ●      ╲────────────────────────╱      ●
    │       ╲     ╱ true f(x) ╲   ╱
    │        ╲───╱              ╲─╱
    │
  ──┼────────────────────────────────── x
   -1                                  1

  Solution: Use Chebyshev nodes instead of equally spaced points!
```

---

## 7. Inverse Interpolation

Lagrange's formula can also find $x$ given $y$: swap the roles of $x$ and $f$ in the formula. Given $(f_0, x_0), (f_1, x_1), \ldots$, interpolate $x$ as a function of $f$.

---

## Summary Table

| Property | Value |
|----------|-------|
| **Formula** | $P_n(x) = \sum_{i=0}^{n} f_i L_i(x)$ |
| **Basis** | $L_i(x) = \prod_{j \neq i} \frac{x - x_j}{x_i - x_j}$ |
| **Spacing** | Any (equally or unequally spaced) |
| **Degree** | At most $n$ for $n+1$ points |
| **Uniqueness** | The interpolating polynomial is unique |
| **Error** | $\frac{f^{(n+1)}(\xi)}{(n+1)!} \prod(x-x_i)$ |

---

## Quick Revision Questions

1. **Write the Lagrange interpolation formula for 4 data points.**

2. **Find the interpolating polynomial through $(0,1), (1,3), (2,7)$ using Lagrange's formula.**

3. **Prove that $\sum_{i=0}^n L_i(x) = 1$.**

4. **What is the Runge phenomenon? How can it be avoided?**

5. **Compare Lagrange interpolation with Newton's divided difference interpolation in terms of computational efficiency.**

6. **How can Lagrange's formula be used for inverse interpolation?**

---

[← Previous: Newton's Backward Difference](03-newton-backward.md) | [Next: Divided Differences →](05-divided-differences.md)
