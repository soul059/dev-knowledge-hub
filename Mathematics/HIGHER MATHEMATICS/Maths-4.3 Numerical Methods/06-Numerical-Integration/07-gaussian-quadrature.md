# Chapter 6.7: Gaussian Quadrature

[← Previous: Romberg Integration](06-romberg-integration.md) | [Next: Unit 7 — Euler's Method →](../07-Ordinary-DEs/01-eulers-method.md)

---

## Overview

**Gaussian quadrature** achieves the highest possible accuracy for a given number of function evaluations by choosing both the **weights** and **nodes** optimally. An $n$-point Gauss formula is exact for polynomials of degree $\leq 2n-1$ — the best possible.

---

## 1. Idea vs Newton-Cotes

| Newton-Cotes | Gaussian Quadrature |
|-------------|---------------------|
| Nodes are equally spaced (fixed) | Nodes are optimally chosen |
| Only weights are "free" | Both nodes AND weights are free |
| $n$ points → exact for degree $\leq n-1$ (or $n$) | $n$ points → exact for degree $\leq 2n-1$ |

With $n$ points, Newton-Cotes has $n$ free parameters (weights), but Gauss has $2n$ (weights + nodes), so it achieves almost double the polynomial exactness.

---

## 2. Gauss-Legendre Quadrature

The most common form. Integral is mapped to $[-1, 1]$:

$$\boxed{\int_{-1}^{1} f(t)\,dt \approx \sum_{i=1}^{n} w_i \, f(t_i)}$$

The nodes $t_i$ are roots of the **Legendre polynomial** $P_n(t)$.

### Standard Nodes and Weights

**2-point ($n=2$)**: Exact for degree $\leq 3$

| $t_i$ | $w_i$ |
|--------|--------|
| $-1/\sqrt{3} \approx -0.57735$ | $1$ |
| $+1/\sqrt{3} \approx +0.57735$ | $1$ |

$$\int_{-1}^{1} f(t)\,dt \approx f(-1/\sqrt{3}) + f(1/\sqrt{3})$$

**3-point ($n=3$)**: Exact for degree $\leq 5$

| $t_i$ | $w_i$ |
|--------|--------|
| $-\sqrt{3/5} \approx -0.77460$ | $5/9 \approx 0.55556$ |
| $0$ | $8/9 \approx 0.88889$ |
| $+\sqrt{3/5} \approx +0.77460$ | $5/9 \approx 0.55556$ |

**4-point ($n=4$)**: Exact for degree $\leq 7$

| $t_i$ | $w_i$ |
|--------|--------|
| $\pm 0.86114$ | $0.34785$ |
| $\pm 0.33998$ | $0.65215$ |

---

## 3. Change of Interval

To integrate over $[a, b]$ instead of $[-1, 1]$, use the substitution:

$$x = \frac{b-a}{2}t + \frac{a+b}{2}, \quad dx = \frac{b-a}{2}dt$$

$$\boxed{\int_a^b f(x)\,dx = \frac{b-a}{2}\int_{-1}^{1} f\left(\frac{b-a}{2}t + \frac{a+b}{2}\right)dt}$$

Then apply the Gauss formula to the transformed integrand.

---

## 4. Worked Example

Evaluate $\int_0^1 e^x\,dx$ using 2-point Gauss-Legendre.

**Step 1**: Change interval from $[0,1]$ to $[-1,1]$:

$$x = \frac{1}{2}t + \frac{1}{2}, \quad dx = \frac{1}{2}dt$$

$$\int_0^1 e^x\,dx = \frac{1}{2}\int_{-1}^{1} e^{(t+1)/2}\,dt$$

**Step 2**: Apply 2-point Gauss ($t_1 = -1/\sqrt{3}, t_2 = 1/\sqrt{3}$, $w_1 = w_2 = 1$):

$$I \approx \frac{1}{2}\left[e^{(-1/\sqrt{3}+1)/2} + e^{(1/\sqrt{3}+1)/2}\right]$$

$$= \frac{1}{2}\left[e^{0.2113} + e^{0.7887}\right]$$

$$= \frac{1}{2}\left[1.2354 + 2.2005\right] = \frac{1}{2}(3.4359) = 1.71795$$

**Exact:** $e - 1 = 1.71828$

**Error:** $0.00033$ — using only **2 function evaluations**! Compare this with 5-point trapezoidal error of $0.0089$.

---

## 5. Why Gauss is So Efficient

```
  Accuracy comparison for n function evaluations:

  Newton-Cotes (n points):  Exact for degree ≤ n-1 (or n)
  Gauss-Legendre (n points): Exact for degree ≤ 2n-1

  Example with n = 3:
    Simpson (3-pt NC): exact for degree ≤ 3
    Gauss (3-pt):      exact for degree ≤ 5  ← MUCH better!
```

---

## 6. Error Formula

For $n$-point Gauss-Legendre:

$$E_n = \frac{2^{2n+1}(n!)^4}{(2n+1)[(2n)!]^3} f^{(2n)}(\xi)$$

For $n=2$: $E = \frac{f^{(4)}(\xi)}{135}$

---

## 7. Other Gaussian Quadrature Variants

| Variant | Weight function | Interval | Use case |
|---------|----------------|----------|----------|
| **Gauss-Legendre** | $w(x) = 1$ | $[-1,1]$ | General integrals |
| **Gauss-Chebyshev** | $w(x) = (1-x^2)^{-1/2}$ | $[-1,1]$ | Singular endpoints |
| **Gauss-Laguerre** | $w(x) = e^{-x}$ | $[0,\infty)$ | Semi-infinite |
| **Gauss-Hermite** | $w(x) = e^{-x^2}$ | $(-\infty,\infty)$ | Full line |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Method** | Optimal nodes and weights |
| **Standard interval** | $[-1, 1]$ |
| **$n$-point exactness** | Degree $\leq 2n-1$ |
| **Nodes** | Roots of Legendre polynomials |
| **2-point nodes** | $\pm 1/\sqrt{3}$, weights $= 1$ |
| **3-point nodes** | $0, \pm\sqrt{3/5}$; weights $8/9, 5/9$ |
| **Interval transform** | $x = \frac{b-a}{2}t + \frac{a+b}{2}$ |

---

## Quick Revision Questions

1. **State the 2-point and 3-point Gauss-Legendre nodes and weights.**

2. **Evaluate $\int_1^3 x^2\,dx$ using 2-point Gauss-Legendre. Verify the result.**

3. **Why is Gaussian quadrature more efficient than Newton-Cotes?**

4. **How do you transform the integral $\int_a^b f(x)\,dx$ to the interval $[-1,1]$?**

5. **For how many function evaluations does Gauss-Legendre match the accuracy of 7-point Newton-Cotes?**

6. **What are the nodes of Gauss-Legendre quadrature and how are they determined?**

---

[← Previous: Romberg Integration](06-romberg-integration.md) | [Next: Unit 7 — Euler's Method →](../07-Ordinary-DEs/01-eulers-method.md)
