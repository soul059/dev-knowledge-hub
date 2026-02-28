# Chapter 2.1: Bisection Method

[← Previous: Stability of Algorithms](../01-Errors-and-Approximations/05-stability-of-algorithms.md) | [Next: Regula Falsi →](02-regula-falsi.md)

---

## Overview

The **Bisection Method** is the simplest and most robust root-finding algorithm. It is based on the **Intermediate Value Theorem**: if $f(a)$ and $f(b)$ have opposite signs and $f$ is continuous on $[a, b]$, then there exists at least one root $c \in (a, b)$ such that $f(c) = 0$.

---

## 1. The Intermediate Value Theorem (IVT)

**Theorem**: If $f$ is continuous on $[a, b]$ and $f(a) \cdot f(b) < 0$, then $\exists \; c \in (a, b)$ such that $f(c) = 0$.

```
  f(x)
    │        f(a) > 0
    │       ●
    │      ╱ ╲
    │     ╱   ╲
    │    ╱     ╲
  ──┼───╱───────╲────────── x
    │  a    c    ╲   b
    │             ╲ ╱
    │              ●
    │           f(b) < 0
    │
    │  f(a)·f(b) < 0  ⟹  root exists in (a,b)
```

---

## 2. Algorithm

### Step-by-Step Procedure

1. **Choose** initial interval $[a, b]$ such that $f(a) \cdot f(b) < 0$
2. **Compute** midpoint: $c = \frac{a + b}{2}$
3. **Evaluate** $f(c)$
4. **Decide**:
   - If $f(c) = 0$ → $c$ is the root. **Stop.**
   - If $f(a) \cdot f(c) < 0$ → root is in $[a, c]$. Set $b = c$.
   - If $f(c) \cdot f(b) < 0$ → root is in $[c, b]$. Set $a = c$.
5. **Check** stopping criterion. If not met, go to Step 2.

### Flowchart

```
  ┌─────────────────────┐
  │  Input: f, a, b, ε  │
  │  f(a)·f(b) < 0?     │
  └──────────┬──────────┘
             │ Yes
             ▼
  ┌─────────────────────┐
  │  c = (a + b) / 2    │◄──────────────────┐
  └──────────┬──────────┘                   │
             │                               │
             ▼                               │
  ┌─────────────────────┐                   │
  │  f(c) = 0?          │──Yes──► DONE: c   │
  └──────────┬──────────┘         is root   │
             │ No                            │
             ▼                               │
  ┌─────────────────────┐                   │
  │  f(a)·f(c) < 0?     │                   │
  └───┬───────────┬─────┘                   │
   Yes│           │No                        │
      ▼           ▼                          │
  ┌───────┐  ┌───────┐                      │
  │ b = c │  │ a = c │                      │
  └───┬───┘  └───┬───┘                      │
      │          │                           │
      ▼          ▼                           │
  ┌─────────────────────┐                   │
  │ |b - a| < ε ?       │──No──────────────┘
  └──────────┬──────────┘
             │ Yes
             ▼
  ┌─────────────────────┐
  │  Root ≈ (a+b)/2     │
  │  OUTPUT c            │
  └─────────────────────┘
```

---

## 3. Convergence Analysis

### Error After $n$ Iterations

After each iteration, the interval is halved:

$$|x_n - r| \leq \frac{b - a}{2^n}$$

where $r$ is the true root and $x_n$ is the midpoint at iteration $n$.

### Number of Iterations Required

To achieve accuracy $\epsilon$:

$$\frac{b - a}{2^n} \leq \epsilon \implies n \geq \frac{\ln(b - a) - \ln \epsilon}{\ln 2} = \log_2\left(\frac{b-a}{\epsilon}\right)$$

### Example: How many iterations for accuracy $10^{-6}$ on $[0, 1]$?

$$n \geq \log_2\left(\frac{1}{10^{-6}}\right) = \log_2(10^6) \approx 19.93$$

So $n = 20$ iterations suffice.

---

## 4. Worked Example

Find a root of $f(x) = x^3 - x - 2$ in $[1, 2]$ to accuracy $0.01$.

**Check**: $f(1) = 1 - 1 - 2 = -2 < 0$, $f(2) = 8 - 2 - 2 = 4 > 0$ ✓

| Iter | $a$ | $b$ | $c = \frac{a+b}{2}$ | $f(c)$ | Sign | New Interval |
|:----:|:----:|:----:|:-------------------:|:------:|:----:|:------------:|
| 1 | 1.0 | 2.0 | 1.5 | $-0.125$ | $-$ | $[1.5, 2.0]$ |
| 2 | 1.5 | 2.0 | 1.75 | $1.609$ | $+$ | $[1.5, 1.75]$ |
| 3 | 1.5 | 1.75 | 1.625 | $0.666$ | $+$ | $[1.5, 1.625]$ |
| 4 | 1.5 | 1.625 | 1.5625 | $0.252$ | $+$ | $[1.5, 1.5625]$ |
| 5 | 1.5 | 1.5625 | 1.53125 | $0.059$ | $+$ | $[1.5, 1.53125]$ |
| 6 | 1.5 | 1.53125 | 1.515625 | $-0.034$ | $-$ | $[1.515625, 1.53125]$ |
| 7 | 1.515625 | 1.53125 | 1.523438 | $0.012$ | $+$ | $[1.515625, 1.523438]$ |

At iteration 7: $|b - a| = 1.523438 - 1.515625 = 0.007813 < 0.01$ ✓

**Root $\approx 1.5195$** (true root: $\approx 1.5214$)

---

## 5. Stopping Criteria

| Criterion | Formula | When to Use |
|-----------|---------|-------------|
| Interval width | $\|b - a\| < \epsilon$ | Most common |
| Function value | $\|f(c)\| < \epsilon$ | When function tolerance matters |
| Relative change | $\|c_n - c_{n-1}\|/\|c_n\| < \epsilon$ | For scale-independent tolerance |
| Max iterations | $n > N_{\max}$ | Safety limit |

---

## 6. Advantages and Disadvantages

| Advantages | Disadvantages |
|-----------|---------------|
| Always converges (guaranteed) | Slow (linear convergence) |
| Simple to implement | Cannot find complex roots |
| Requires only function evaluations | Cannot find roots where $f$ touches zero without crossing |
| Provides error bound at each step | Needs initial bracketing $[a,b]$ with sign change |
| No derivatives needed | Converges to only one root per interval |

---

## 7. Convergence Rate

The bisection method has **linear convergence** with rate $1/2$:

$$e_{n+1} \leq \frac{1}{2} e_n$$

where $e_n = |x_n - r|$ is the error at iteration $n$.

```
  Convergence comparison:

  Iteration    Bisection Error     Newton Error
  ─────────    ───────────────     ────────────
      0          1.0                1.0
      1          0.5                0.1    (quadratic)
      2          0.25               0.001
      3          0.125              0.000001
      4          0.0625             10⁻¹²
      5          0.03125            10⁻²⁴

  Bisection: ~3.3 iterations per decimal digit
  Newton: digits roughly double each iteration
```

---

## 8. Real-World Applications

- **Engineering**: Finding operating points in nonlinear circuits
- **Finance**: Computing interest rates that satisfy present-value equations
- **Physics**: Finding energy eigenvalues in quantum mechanics
- **Computer Graphics**: Ray-surface intersection (bisecting parameter interval)

---

## Summary Table

| Property | Value |
|----------|-------|
| **Type** | Bracketing method |
| **Requires** | Continuous $f$, initial $[a,b]$ with $f(a)f(b) < 0$ |
| **Formula** | $c = (a+b)/2$ |
| **Convergence** | Linear, guaranteed |
| **Rate** | Error halves each iteration |
| **Iterations for $\epsilon$** | $n = \lceil \log_2((b-a)/\epsilon) \rceil$ |
| **Derivatives needed** | No |
| **Multiple roots** | Fails (no sign change) |

---

## Quick Revision Questions

1. **State the Intermediate Value Theorem and explain its role in the bisection method.**

2. **Find the number of iterations needed to find a root in $[1, 3]$ to accuracy $10^{-8}$.**

3. **Apply bisection to find a root of $f(x) = x^2 - 5$ in $[2, 3]$. Perform 4 iterations.**

4. **Why can't the bisection method find a root of $f(x) = x^2$ in $[-1, 1]$?**

5. **Compare the convergence of bisection with Newton-Raphson. When would you prefer bisection?**

6. **What happens if $f$ is discontinuous in $[a, b]$? Does the method still work?**

---

[← Previous: Stability of Algorithms](../01-Errors-and-Approximations/05-stability-of-algorithms.md) | [Next: Regula Falsi →](02-regula-falsi.md)
