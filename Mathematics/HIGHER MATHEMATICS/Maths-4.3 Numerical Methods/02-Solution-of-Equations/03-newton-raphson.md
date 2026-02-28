# Chapter 2.3: Newton-Raphson Method

[← Previous: Regula Falsi](02-regula-falsi.md) | [Next: Secant Method →](04-secant-method.md)

---

## Overview

The **Newton-Raphson method** (or simply Newton's method) is the most important root-finding algorithm in numerical analysis. It uses the function value **and its derivative** to construct a tangent line at the current guess, then takes the x-intercept of that tangent as the next guess. Under good conditions, it converges **quadratically** — the number of correct digits roughly doubles at each step.

---

## 1. Geometric Idea

```
  f(x)
    │
    │          ● (xₙ, f(xₙ))
    │         ╱│
    │        ╱ │  Tangent line at xₙ
    │       ╱  │
    │      ╱   │
    │     ╱    │
  ──┼────╳─────┼──────── x
    │   xₙ₊₁  xₙ
    │
    │  xₙ₊₁ = xₙ - f(xₙ)/f'(xₙ)
    │
    │  The tangent crosses x-axis at xₙ₊₁
```

---

## 2. Derivation

Starting from Taylor expansion of $f(x)$ about $x_n$:

$$f(x) \approx f(x_n) + f'(x_n)(x - x_n)$$

Setting $f(x) = 0$ and solving for $x$:

$$0 = f(x_n) + f'(x_n)(x - x_n)$$

$$\boxed{x_{n+1} = x_n - \frac{f(x_n)}{f'(x_n)}}$$

This is the **Newton-Raphson iteration formula**.

---

## 3. Algorithm

```
  ┌──────────────────────────┐
  │  Input: f, f', x₀, ε    │
  └────────────┬─────────────┘
               ▼
  ┌──────────────────────────┐
  │  Compute f(xₙ), f'(xₙ)  │◄──────────┐
  └────────────┬─────────────┘           │
               ▼                          │
  ┌──────────────────────────┐           │
  │  f'(xₙ) ≈ 0 ?           │─Yes─►FAIL │
  └────────────┬─────────────┘  (near    │
               │ No              stationary│
               ▼                  point)  │
  ┌──────────────────────────┐           │
  │  xₙ₊₁ = xₙ - f(xₙ)     │           │
  │              / f'(xₙ)    │           │
  └────────────┬─────────────┘           │
               ▼                          │
  ┌──────────────────────────┐           │
  │ |xₙ₊₁ - xₙ| < ε        │           │
  │  or |f(xₙ₊₁)| < ε ?    │           │
  └───┬──────────────┬──────┘           │
    Yes              No ─────────────────┘
      ▼
  ┌──────────┐
  │ Root ≈ xₙ₊₁│
  └──────────┘
```

---

## 4. Convergence Analysis

### Order of Convergence

For a simple root ($f'(r) \neq 0$), Newton's method has **quadratic convergence**:

$$e_{n+1} = \frac{f''(c)}{2f'(c)} \cdot e_n^2$$

where $e_n = |x_n - r|$ and $c$ is between $x_n$ and $r$.

This means: if the error is $10^{-3}$, next iteration gives $\sim 10^{-6}$, then $\sim 10^{-12}$.

### Convergence Table

```
  Iteration    Error              Digits Correct
  ─────────    ──────────────     ──────────────
     0         10⁻¹               1
     1         10⁻²               2
     2         10⁻⁴               4
     3         10⁻⁸               8
     4         10⁻¹⁶              16  (machine precision!)
```

Only 4-5 iterations typically needed for full machine precision!

---

## 5. Worked Example

Find $\sqrt{2}$ using Newton-Raphson. Let $f(x) = x^2 - 2$, $f'(x) = 2x$.

$$x_{n+1} = x_n - \frac{x_n^2 - 2}{2x_n} = \frac{x_n + 2/x_n}{2}$$

Starting with $x_0 = 1$:

| Iter | $x_n$ | $f(x_n)$ | $f'(x_n)$ | $x_{n+1}$ | Error |
|:----:|:------:|:--------:|:---------:|:---------:|:-----:|
| 0 | 1.000000 | $-1.000$ | 2.000 | 1.500000 | 0.0858 |
| 1 | 1.500000 | $0.250$ | 3.000 | 1.416667 | 0.0025 |
| 2 | 1.416667 | $0.00694$ | 2.833 | 1.414216 | 0.000002 |
| 3 | 1.414216 | $5.4 \times 10^{-6}$ | 2.828 | 1.414214 | $10^{-12}$ |

True value: $\sqrt{2} = 1.41421356...$

**Quadratic convergence confirmed**: errors are roughly squared at each step.

---

## 6. When Newton's Method Fails

### Case 1: Zero Derivative

If $f'(x_n) = 0$, the tangent is horizontal — no x-intercept exists.

```
  f(x)
    │     ● (xₙ, f(xₙ))
    │─────────────────────  horizontal tangent
    │                        f'(xₙ) = 0
  ──┼──────────────────── x
    │ No intersection!
```

### Case 2: Cycling

```
  f(x)
    │        ╱╲
    │       ╱  ╲
    │ ●────╱    ╲────● 
    │ x₀           x₁    Cycles back and forth!
  ──┼──────────────────── x
    │      ╲    ╱
    │       ╲  ╱
    │        ╲╱
```

### Case 3: Divergence (Bad Initial Guess)

The iterates move away from the root if starting too far.

### Case 4: Multiple Roots

For a root of multiplicity $m > 1$ (i.e., $f(r) = f'(r) = \cdots = f^{(m-1)}(r) = 0$):

- Convergence degrades to **linear** with rate $\frac{m-1}{m}$
- Fix: **Modified Newton**: $x_{n+1} = x_n - m\frac{f(x_n)}{f'(x_n)}$

---

## 7. Sufficient Conditions for Convergence

**Theorem**: Newton's method converges if:
1. $f$, $f'$, $f''$ are continuous near the root $r$
2. $f'(r) \neq 0$ (simple root)
3. $x_0$ is "sufficiently close" to $r$

More precisely, convergence is guaranteed if:

$$|x_0 - r| < \frac{2|f'(r)|}{M}$$

where $M = \max |f''(x)|$ in a neighborhood of $r$.

---

## 8. Newton's Method for Common Problems

| Problem | $f(x)$ | Iteration Formula |
|---------|--------|-------------------|
| $\sqrt{a}$ | $x^2 - a$ | $x_{n+1} = \frac{1}{2}\left(x_n + \frac{a}{x_n}\right)$ |
| $\frac{1}{a}$ (no division) | $\frac{1}{x} - a$ | $x_{n+1} = x_n(2 - ax_n)$ |
| $\sqrt[3]{a}$ | $x^3 - a$ | $x_{n+1} = \frac{1}{3}\left(2x_n + \frac{a}{x_n^2}\right)$ |
| $a^{1/p}$ | $x^p - a$ | $x_{n+1} = \frac{1}{p}\left((p-1)x_n + \frac{a}{x_n^{p-1}}\right)$ |

---

## 9. Newton's Method in Higher Dimensions

For a system $\mathbf{F}(\mathbf{x}) = \mathbf{0}$, the iteration becomes:

$$\mathbf{x}_{n+1} = \mathbf{x}_n - \mathbf{J}^{-1}(\mathbf{x}_n) \cdot \mathbf{F}(\mathbf{x}_n)$$

where $\mathbf{J}$ is the **Jacobian matrix**:

$$J_{ij} = \frac{\partial F_i}{\partial x_j}$$

---

## 10. Real-World Applications

| Application | Details |
|-------------|---------|
| Optimization algorithms | Finding critical points where gradient = 0 |
| Circuit simulation (SPICE) | Solving nonlinear circuit equations |
| Structural analysis | Nonlinear stress-strain problems |
| Machine learning | Training neural networks (second-order methods) |
| Robotics | Inverse kinematics |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Type** | Open method (no bracket) |
| **Formula** | $x_{n+1} = x_n - f(x_n)/f'(x_n)$ |
| **Convergence order** | 2 (quadratic) for simple roots |
| **Requires** | $f(x)$ and $f'(x)$ |
| **Cost per iteration** | 1 function + 1 derivative evaluation |
| **Convergence** | Not guaranteed; depends on initial guess |
| **Multiple roots** | Degrades to linear; use modified formula |
| **Failure modes** | $f'(x_n) = 0$, bad initial guess, cycling |

---

## Quick Revision Questions

1. **Derive the Newton-Raphson formula from the Taylor series expansion.**

2. **Apply Newton-Raphson to find $\sqrt{5}$. Start with $x_0 = 2$ and do 3 iterations.**

3. **Show that Newton's method has quadratic convergence for simple roots.**

4. **What happens when Newton's method is applied to $f(x) = x^3$ (a triple root at $x = 0$)? What is the convergence rate?**

5. **Give two different scenarios where Newton's method fails to converge.**

6. **How would you modify Newton's method for a root of multiplicity $m$?**

---

[← Previous: Regula Falsi](02-regula-falsi.md) | [Next: Secant Method →](04-secant-method.md)
