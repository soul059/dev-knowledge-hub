# Chapter 6.1: Trapezoidal Rule

[← Previous: Error Estimation in Differentiation](../05-Numerical-Differentiation/05-error-estimation.md) | [Next: Simpson's 1/3 Rule →](02-simpsons-one-third.md)

---

## Overview

The **Trapezoidal Rule** approximates a definite integral by dividing the area under the curve into trapezoids. It is the simplest Newton-Cotes formula and provides $O(h^2)$ accuracy.

---

## 1. Single-Interval Formula

$$\boxed{\int_a^b f(x)\,dx \approx \frac{h}{2}[f(a) + f(b)]}$$

where $h = b - a$.

### Geometric Interpretation

```
  f(x)
    │
    │   f(a)●─────────────●f(b)
    │      ╱│             │╲
    │     ╱ │  Trapezoid  │ ╲  ← True curve
    │    ╱  │             │  ╲
    │   ╱   │             │
    │  ╱    │             │
    └──┼────┼─────────────┼──── x
       a                  b
```

The function is replaced by a straight line (linear interpolation).

---

## 2. Composite Trapezoidal Rule

Divide $[a,b]$ into $n$ equal subintervals: $h = \frac{b-a}{n}$, $x_i = a + ih$.

$$\boxed{\int_a^b f(x)\,dx \approx \frac{h}{2}\left[f_0 + 2\sum_{i=1}^{n-1} f_i + f_n\right]}$$

```
  f(x)
    │    ●─────●
    │   ╱│╲   ╱│╲
    │  ╱ │ ╲ ╱ │ ╲    ●
    │ ╱  │  ●  │  ╲  ╱│
    ●    │  │  │   ╲╱ │
    │    │  │  │   │  │
    └────┼──┼──┼───┼──┼── x
         x₀ x₁ x₂ x₃ x₄
         ↑               ↑
      weight 1    weight 1 (endpoints)
            weight 2 (interior points)
```

---

## 3. Error Analysis

### Single interval

$$E_T = -\frac{h^3}{12}f''(\xi), \quad \xi \in (a,b)$$

### Composite rule ($n$ intervals)

$$\boxed{E_T = -\frac{(b-a)h^2}{12}f''(\xi) = -\frac{(b-a)^3}{12n^2}f''(\xi)}$$

Key facts:
- Error is $O(h^2)$ — doubles accuracy means quadrupling $n$.
- Error depends on $f''$ — **exact for linear functions** ($f'' = 0$).

---

## 4. Worked Example

Evaluate $\int_0^1 e^x \, dx$ using the trapezoidal rule with $n = 4$.

$h = (1-0)/4 = 0.25$

| $i$ | $x_i$ | $f(x_i) = e^{x_i}$ |
|-----|--------|---------------------|
| 0 | 0.00 | 1.0000 |
| 1 | 0.25 | 1.2840 |
| 2 | 0.50 | 1.6487 |
| 3 | 0.75 | 2.1170 |
| 4 | 1.00 | 2.7183 |

$$I \approx \frac{0.25}{2}\left[1.0000 + 2(1.2840 + 1.6487 + 2.1170) + 2.7183\right]$$

$$= 0.125\left[1.0000 + 2(5.0497) + 2.7183\right]$$

$$= 0.125\left[1.0000 + 10.0994 + 2.7183\right]$$

$$= 0.125 \times 13.8177 = 1.7272$$

**Exact value:** $e^1 - e^0 = 1.7183$

**Error:** $1.7272 - 1.7183 = 0.0089$

---

## 5. Error Estimation without $f''$

If $f''$ is unknown, use two estimates:

$$I_n \approx I + c \cdot h^2 \quad \text{and} \quad I_{2n} \approx I + c \cdot (h/2)^2$$

Then: $\quad I \approx I_{2n} + \frac{I_{2n} - I_n}{3}$ (Romberg's idea)

---

## 6. Derivation via Taylor Series

Expand $f(x)$ around $a$ in the single interval $[a, a+h]$:

$$\int_a^{a+h} f(x)\,dx = hf(a) + \frac{h^2}{2}f'(a) + \frac{h^3}{6}f''(a) + \cdots$$

The trapezoidal approximation:

$$\frac{h}{2}[f(a) + f(a+h)] = hf(a) + \frac{h^2}{2}f'(a) + \frac{h^2}{4} \cdot 2f'(a) + \cdots$$

Wait, more carefully:

$$\frac{h}{2}[f(a) + f(a+h)] = \frac{h}{2}\left[2f(a) + hf'(a) + \frac{h^2}{2}f''(a) + \cdots\right]$$

$$= hf(a) + \frac{h^2}{2}f'(a) + \frac{h^3}{4}f''(a) + \cdots$$

Subtracting: Error $= \frac{h^3}{4}f''(a) - \frac{h^3}{6}f''(a) = \frac{h^3}{12}f''(a)$

This confirms $E_T = -\frac{h^3}{12}f''(\xi)$.

---

## Summary Table

| Property | Value |
|----------|-------|
| **Type** | Closed Newton-Cotes, degree 1 |
| **Formula** | $\frac{h}{2}[f_0 + 2\sum f_i + f_n]$ |
| **Error (composite)** | $-\frac{(b-a)h^2}{12}f''(\xi)$ |
| **Order** | $O(h^2)$ |
| **Exact for** | Polynomials of degree $\leq 1$ |
| **Advantage** | Simple, easy to implement |
| **Disadvantage** | Low accuracy compared to Simpson's |

---

## Quick Revision Questions

1. **State the composite trapezoidal rule and its error term.**

2. **Evaluate $\int_0^6 f(x)\,dx$ using the trapezoidal rule with: $f(0)=1, f(2)=5, f(4)=7, f(6)=3$.**

3. **How many subintervals are needed to evaluate $\int_0^1 e^x\,dx$ with error $< 10^{-4}$ using the trapezoidal rule?**

4. **Prove the trapezoidal rule is exact for linear functions.**

5. **Explain why doubling $n$ reduces the error by a factor of 4.**

6. **How can two trapezoidal estimates be combined to get a better estimate (Romberg)?**

---

[← Previous: Error Estimation in Differentiation](../05-Numerical-Differentiation/05-error-estimation.md) | [Next: Simpson's 1/3 Rule →](02-simpsons-one-third.md)
