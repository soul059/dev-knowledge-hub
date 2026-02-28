# Chapter 6.2: Simpson's 1/3 Rule

[← Previous: Trapezoidal Rule](01-trapezoidal-rule.md) | [Next: Simpson's 3/8 Rule →](03-simpsons-three-eighth.md)

---

## Overview

**Simpson's 1/3 Rule** fits a **parabola** (degree-2 polynomial) through three equally spaced points and integrates it exactly. It is the most widely used numerical integration formula because of its excellent accuracy-to-effort ratio.

---

## 1. Single-Interval Formula

Over $[a, b]$ with midpoint $m = (a+b)/2$ and $h = (b-a)/2$:

$$\boxed{\int_a^b f(x)\,dx \approx \frac{h}{3}[f(a) + 4f(m) + f(b)]}$$

Or equivalently, with $h = (b-a)/2$, $x_0 = a$, $x_1 = m$, $x_2 = b$:

$$\int_{x_0}^{x_2} f(x)\,dx \approx \frac{h}{3}[f_0 + 4f_1 + f_2]$$

---

## 2. Geometric Interpretation

```
  f(x)
    │        ●  f₁ (midpoint)
    │       ╱ ╲
    │      ╱   ╲
    │  f₀ ●     ╲    ● f₂
    │    ╱  Parabola  ╲
    │   ╱    fit       ╲
    │  ╱                ╲
    └──┼────────┼────────┼── x
       x₀      x₁       x₂
       ←── h ──→←── h ──→
```

The function is approximated by a **parabola** through $(x_0,f_0), (x_1,f_1), (x_2,f_2)$.

---

## 3. Composite Simpson's 1/3 Rule

Divide $[a,b]$ into $n$ subintervals ($n$ **must be even**), $h = (b-a)/n$:

$$\boxed{\int_a^b f(x)\,dx \approx \frac{h}{3}\left[f_0 + 4\sum_{\text{odd } i} f_i + 2\sum_{\text{even } i} f_i + f_n\right]}$$

**Weight pattern**: $1, 4, 2, 4, 2, 4, 2, \ldots, 4, 2, 4, 1$

```
  Weights:  1    4    2    4    2    4    1
            │    │    │    │    │    │    │
            f₀   f₁   f₂   f₃   f₄   f₅   f₆
            ←─────── n = 6 intervals ──────→
```

---

## 4. Error Analysis

### Single interval

$$E_S = -\frac{h^5}{90}f^{(4)}(\xi)$$

### Composite rule ($n$ intervals)

$$\boxed{E_S = -\frac{(b-a)h^4}{180}f^{(4)}(\xi) = -\frac{(b-a)^5}{180 \, n^4}f^{(4)}(\xi)}$$

Key facts:
- Error is $O(h^4)$ — **two orders better** than trapezoidal!
- Exact for polynomials of degree $\leq 3$ (not just $\leq 2$; this is a "bonus").
- The $f^{(4)}$ dependence is why cubics are also exact.

---

## 5. Worked Example

Evaluate $\int_0^1 e^x \, dx$ using Simpson's 1/3 rule with $n = 4$.

$h = 0.25$

| $i$ | $x_i$ | $f_i = e^{x_i}$ | Weight |
|-----|--------|-----------------|--------|
| 0 | 0.00 | 1.0000 | 1 |
| 1 | 0.25 | 1.2840 | 4 |
| 2 | 0.50 | 1.6487 | 2 |
| 3 | 0.75 | 2.1170 | 4 |
| 4 | 1.00 | 2.7183 | 1 |

$$I \approx \frac{0.25}{3}\left[1.0000 + 4(1.2840 + 2.1170) + 2(1.6487) + 2.7183\right]$$

$$= \frac{0.25}{3}\left[1.0000 + 4(3.4010) + 3.2974 + 2.7183\right]$$

$$= \frac{0.25}{3}\left[1.0000 + 13.6040 + 3.2974 + 2.7183\right]$$

$$= \frac{0.25}{3} \times 20.6197 = \frac{5.1549}{3} = 0.08333 \times 20.6197$$

Wait, let me recalculate:

$$= \frac{0.25}{3} \times 20.6197 = 0.08333 \times 20.6197 = 1.71831$$

**Exact value:** $e - 1 = 1.71828$

**Error:** $0.00003$ — much better than the trapezoidal result ($0.0089$)!

---

## 6. Why Simpson's Rule is Exact for Cubics

By symmetry: $\int_{-h}^{h} x^3 \, dx = 0$ (odd function over symmetric interval).

Simpson with $f(-h) = -h^3$, $f(0) = 0$, $f(h) = h^3$:

$$\frac{h}{3}[-h^3 + 4(0) + h^3] = 0 \quad \checkmark$$

The odd-power terms cancel due to the symmetric weight pattern, giving a "free" order of accuracy.

---

## 7. Requirement: $n$ Must Be Even

Simpson's 1/3 rule requires **pairs** of subintervals (each parabola spans 2 subintervals). If $n$ is odd:
- Use Simpson's 3/8 rule for 3 subintervals + Simpson's 1/3 for the rest
- Or use the trapezoidal rule for one interval

---

## Summary Table

| Property | Value |
|----------|-------|
| **Type** | Closed Newton-Cotes, degree 2 |
| **Formula** | $\frac{h}{3}[f_0 + 4f_1 + 2f_2 + 4f_3 + \cdots + f_n]$ |
| **Requirement** | $n$ must be **even** |
| **Error** | $-\frac{(b-a)h^4}{180}f^{(4)}(\xi)$ |
| **Order** | $O(h^4)$ |
| **Exact for** | Polynomials of degree $\leq 3$ |
| **vs Trapezoidal** | $O(h^4)$ vs $O(h^2)$ — far superior |

---

## Quick Revision Questions

1. **State the composite Simpson's 1/3 rule and identify the weight pattern.**

2. **Evaluate $\int_0^6 f(x)\,dx$ using Simpson's rule with $f(0)=1, f(1)=4, f(2)=9, f(3)=16, f(4)=25, f(5)=36, f(6)=49$.**

3. **Why must $n$ be even for Simpson's 1/3 rule?**

4. **Prove that Simpson's rule is exact for $f(x) = x^3$.**

5. **How many subintervals are needed to evaluate $\int_0^1 e^x\,dx$ with error $< 10^{-6}$?**

6. **Compare the errors of trapezoidal and Simpson's rules applied to $\int_0^1 x^2\,dx$ with $n=2$.**

---

[← Previous: Trapezoidal Rule](01-trapezoidal-rule.md) | [Next: Simpson's 3/8 Rule →](03-simpsons-three-eighth.md)
