# Chapter 2.4: Secant Method

[← Previous: Newton-Raphson](03-newton-raphson.md) | [Next: Fixed-Point Iteration →](05-fixed-point-iteration.md)

---

## Overview

The **Secant Method** is a derivative-free alternative to Newton-Raphson. Instead of computing $f'(x_n)$ analytically, it approximates the derivative using two previous iterates. This makes it practical when the derivative is expensive or unavailable.

---

## 1. Derivation

Replace the derivative in Newton's formula with a finite-difference approximation:

$$f'(x_n) \approx \frac{f(x_n) - f(x_{n-1})}{x_n - x_{n-1}}$$

Substituting into $x_{n+1} = x_n - \frac{f(x_n)}{f'(x_n)}$:

$$\boxed{x_{n+1} = x_n - f(x_n) \cdot \frac{x_n - x_{n-1}}{f(x_n) - f(x_{n-1})}}$$

Or equivalently:

$$x_{n+1} = \frac{x_{n-1} f(x_n) - x_n f(x_{n-1})}{f(x_n) - f(x_{n-1})}$$

---

## 2. Geometric Interpretation

```
  f(x)
    │
    │    ● (xₙ₋₁, f(xₙ₋₁))
    │     ╲
    │      ╲  Secant line through
    │       ╲  both points
    │        ╲
  ──┼─────────╳────────●────── x
    │       xₙ₊₁    (xₙ, f(xₙ))
    │                 ╱
    │                ╱
    │
    │  Compare:
    │  Newton uses TANGENT at xₙ (needs f')
    │  Secant uses line through (xₙ₋₁,f(xₙ₋₁)) and (xₙ,f(xₙ))
```

---

## 3. Algorithm

```
  ┌────────────────────────────┐
  │  Input: f, x₀, x₁, ε     │
  └─────────────┬──────────────┘
                ▼
  ┌────────────────────────────┐
  │ Compute f(x₀), f(x₁)     │
  └─────────────┬──────────────┘
                ▼
  ┌────────────────────────────┐
  │ x₂ = x₁ - f(x₁)·         │◄──────┐
  │    (x₁-x₀)/(f(x₁)-f(x₀)) │       │
  └─────────────┬──────────────┘       │
                ▼                       │
  ┌────────────────────────────┐       │
  │ |x₂ - x₁| < ε ?          │       │
  └───┬────────────────┬──────┘       │
    Yes                No              │
      ▼                │               │
  ┌────────┐          ▼               │
  │Root ≈ x₂│   x₀=x₁, x₁=x₂ ──────┘
  └────────┘
```

---

## 4. Worked Example

Find a root of $f(x) = x^3 - x - 2$ with $x_0 = 1$, $x_1 = 2$.

| Iter | $x_{n-1}$ | $x_n$ | $f(x_{n-1})$ | $f(x_n)$ | $x_{n+1}$ | $\|x_{n+1}-x_n\|$ |
|:----:|:---------:|:-----:|:------------:|:--------:|:---------:|:-----------------:|
| 1 | 1.0 | 2.0 | $-2$ | $4$ | 1.3333 | 0.6667 |
| 2 | 2.0 | 1.3333 | $4$ | $-0.963$ | 1.4545 | 0.1212 |
| 3 | 1.3333 | 1.4545 | $-0.963$ | $-0.379$ | 1.5685 | 0.1140 |
| 4 | 1.4545 | 1.5685 | $-0.379$ | $0.293$ | 1.5168 | 0.0517 |
| 5 | 1.5685 | 1.5168 | $0.293$ | $-0.027$ | 1.5209 | 0.0041 |
| 6 | 1.5168 | 1.5209 | $-0.027$ | $-0.003$ | 1.5214 | 0.0005 |

Root $\approx 1.5214$ ✓

---

## 5. Convergence

The secant method has **superlinear convergence** with order:

$$p = \frac{1 + \sqrt{5}}{2} \approx 1.618 \quad \text{(the golden ratio!)}$$

The error relation:

$$e_{n+1} \approx C \cdot e_n^{1.618}$$

### Comparison of Convergence Orders

| Method | Order | Evaluations/Iter | Efficiency Index |
|--------|:-----:|:----------------:|:----------------:|
| Bisection | 1.0 | 1 | 1.0 |
| Regula Falsi | ~1.618 | 1 | 1.618 |
| Secant | 1.618 | 1 | 1.618 |
| Newton-Raphson | 2.0 | 2 ($f$ and $f'$) | $\sqrt{2} \approx 1.414$ |

**Key insight**: Per function evaluation, the secant method is actually **more efficient** than Newton-Raphson!

The **efficiency index** $= p^{1/k}$ where $p$ is the order and $k$ is function evaluations per step.

---

## 6. Secant vs. Newton vs. Regula Falsi

```
  ┌──────────────────────────────────────────────────────┐
  │                    COMPARISON                         │
  ├──────────────┬──────────┬──────────┬────────────────┤
  │ Feature      │ Newton   │ Secant   │ Regula Falsi   │
  ├──────────────┼──────────┼──────────┼────────────────┤
  │ Requires f'  │ Yes      │ No       │ No             │
  │ Initial pts  │ 1        │ 2        │ 2              │
  │ Brackets root│ No       │ No       │ Yes            │
  │ Convergence  │ Quadratic│ 1.618    │ ~1.618*        │
  │ Can diverge  │ Yes      │ Yes      │ No             │
  │ Stagnation   │ No       │ No       │ Yes            │
  └──────────────┴──────────┴──────────┴────────────────┘
  * Regula Falsi can slow down due to stagnation
```

---

## 7. Advantages and Disadvantages

| Advantages | Disadvantages |
|-----------|---------------|
| No derivative needed | Not guaranteed to converge |
| Fast (superlinear) convergence | Requires two starting points |
| One function evaluation per step | Can diverge with bad initial guesses |
| Higher efficiency index than Newton | No bracket — can miss roots |

---

## 8. Real-World Applications

| Application | Why Secant Method |
|-------------|------------------|
| Black-box functions | When $f'$ is unknown or costly |
| Experimental data | Derivative not available from data |
| Legacy code | When only function values can be obtained |
| Optimization | Line search in quasi-Newton methods |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Type** | Open method |
| **Formula** | $x_{n+1} = x_n - f(x_n)\frac{x_n - x_{n-1}}{f(x_n) - f(x_{n-1})}$ |
| **Convergence order** | $\phi = (1+\sqrt{5})/2 \approx 1.618$ |
| **Requires** | $f(x)$ only; two initial guesses |
| **Cost per iteration** | 1 function evaluation |
| **Convergence** | Not guaranteed |
| **Efficiency index** | $1.618$ (better than Newton's $\sqrt{2}$) |

---

## Quick Revision Questions

1. **How is the secant method derived from Newton-Raphson?**

2. **Apply the secant method to $f(x) = e^x - 3x$ with $x_0 = 0$, $x_1 = 1$. Perform 3 iterations.**

3. **What is the order of convergence of the secant method? How does it compare to Newton-Raphson in terms of efficiency index?**

4. **Why is the secant method NOT a bracketing method? What are the consequences?**

5. **Compare the geometric interpretations of Newton-Raphson and the secant method with diagrams.**

6. **Can the secant method converge faster than Newton-Raphson? Justify with efficiency index analysis.**

---

[← Previous: Newton-Raphson](03-newton-raphson.md) | [Next: Fixed-Point Iteration →](05-fixed-point-iteration.md)
