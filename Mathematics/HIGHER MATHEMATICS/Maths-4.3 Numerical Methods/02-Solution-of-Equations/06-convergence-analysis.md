# Chapter 2.6: Convergence Analysis

[← Previous: Fixed-Point Iteration](05-fixed-point-iteration.md) | [Next: Unit 3 — Gaussian Elimination →](../03-System-of-Equations/01-gaussian-elimination.md)

---

## Overview

This chapter compares all root-finding methods studied so far, discussing their convergence orders, rates, efficiency, and practical selection criteria. Understanding convergence analysis helps you choose the right method for each problem.

---

## 1. Order of Convergence

A sequence $\{x_n\}$ converging to $r$ has **order of convergence** $p$ if:

$$\lim_{n \to \infty} \frac{|x_{n+1} - r|}{|x_n - r|^p} = C \neq 0$$

| Order $p$ | Name | Error Behavior |
|:---------:|------|----------------|
| 1 | Linear | $e_{n+1} \approx C \cdot e_n$ |
| 1 < p < 2 | Superlinear | $e_{n+1} \approx C \cdot e_n^p$ |
| 2 | Quadratic | $e_{n+1} \approx C \cdot e_n^2$ |
| 3 | Cubic | $e_{n+1} \approx C \cdot e_n^3$ |

### Intuitive Understanding

```
  Digits of accuracy gained per iteration:

  Linear (p=1, C=0.5):
  iter:  0    1    2    3    4    5    6    7    8    9   10
  digits: 0    0    1    1    1    2    2    2    3    3    3
  (about 3 iterations per new digit)

  Quadratic (p=2):
  iter:  0    1    2    3     4
  digits: 0    1    2    4     8
  (digits DOUBLE each iteration)

  Cubic (p=3):
  iter:  0    1    2    3
  digits: 0    1    3    9
  (digits TRIPLE each iteration)
```

---

## 2. Master Comparison Table

| Method | Order $p$ | Rate $C$ | $f$ evals/iter | $f'$ needed? | Bracket? | Guaranteed? |
|--------|:---------:|:--------:|:--------------:|:------------:|:--------:|:-----------:|
| Bisection | 1 | 0.5 | 1 | No | Yes | Yes |
| Regula Falsi | $\approx 1.618^*$ | — | 1 | No | Yes | Yes |
| Newton-Raphson | 2 | $\frac{f''}{2f'}$ | 1+1 | Yes | No | No |
| Secant | 1.618 | — | 1 | No | No | No |
| Fixed-Point | 1 (or higher) | $\|g'(r)\|$ | 1 | Depends | No | Conditional |
| Modified Newton | 2 | — | 1+1+1 | Yes+$f''$ | No | No |

$^*$ Regula Falsi can degrade to linear due to stagnation.

---

## 3. Efficiency Index

The **efficiency index** $EI$ accounts for computational cost:

$$EI = p^{1/k}$$

where $p$ = convergence order, $k$ = function evaluations per iteration.

| Method | $p$ | $k$ | $EI = p^{1/k}$ |
|--------|:---:|:---:|:---------------:|
| Bisection | 1 | 1 | $1.000$ |
| Secant | 1.618 | 1 | $1.618$ |
| Newton | 2 | 2 | $\sqrt{2} = 1.414$ |
| Müller | 1.84 | 1 | $1.840$ |

**Secant method wins** on efficiency per function evaluation!

---

## 4. Convergence Speed Comparison

### Practical Example: Finding $\sqrt{2}$, i.e., root of $f(x) = x^2 - 2$

Starting from similar conditions, number of iterations to reach $10^{-10}$ accuracy:

```
  Method            Iterations    Function Evaluations
  ─────────────     ──────────    ────────────────────
  Bisection            34              34
  Regula Falsi         12              12
  Secant                8               8
  Newton-Raphson        5              10 (f + f')
```

```
  Error vs. Iteration:

  log₁₀(error)
    0 ┤ B  R  S  N
      │ ╲  ╲  ╲  ╲
   -2 ┤  B  ╲  ╲  ╲
      │   ╲  R  ╲   N
   -4 ┤    B  ╲  S    ╲
      │     ╲  R  ╲     N
   -6 ┤      B  ╲  S      ╲
      │       ╲  R   ╲       N
   -8 ┤        B  ╲    S
      │         ╲  R     ╲
  -10 ┤          B   R     S    N
      ├──┤──┤──┤──┤──┤──┤──┤──┤──┤
      0  2  4  6  8  10 12 14 16

  B = Bisection, R = Regula Falsi
  S = Secant, N = Newton-Raphson
```

---

## 5. Multiple Roots and Their Effect

When $f(x) = (x - r)^m \cdot q(x)$ with $q(r) \neq 0$, the root $r$ has **multiplicity** $m$.

### Effect on Convergence

| Method | Simple Root ($m=1$) | Multiple Root ($m>1$) |
|--------|:------------------:|:--------------------:|
| Newton | Quadratic ($p=2$) | Linear ($p=1$, rate $\frac{m-1}{m}$) |
| Secant | $p = 1.618$ | $p = 1$ |
| Bisection | Linear (rate 0.5) | Fails (no sign change if $m$ even) |

### Fix: Modified Newton for Multiple Roots

$$x_{n+1} = x_n - m \cdot \frac{f(x_n)}{f'(x_n)}$$

Or use $u(x) = f(x)/f'(x)$ which has only simple roots:

$$x_{n+1} = x_n - \frac{u(x_n)}{u'(x_n)} = x_n - \frac{f(x_n) f'(x_n)}{[f'(x_n)]^2 - f(x_n) f''(x_n)}$$

---

## 6. When to Use Which Method

```
  ┌─────────────────────────────────────────────────────┐
  │            METHOD SELECTION GUIDE                    │
  │                                                      │
  │  Is f' available and cheap?                          │
  │    ├─ YES: Is good initial guess available?          │
  │    │   ├─ YES → Newton-Raphson (fastest)            │
  │    │   └─ NO  → Bisection first, then Newton        │
  │    └─ NO:  Do you have bracket [a,b]?               │
  │        ├─ YES → Regula Falsi or Bisection           │
  │        └─ NO  → Secant method                       │
  │                                                      │
  │  Is guaranteed convergence needed?                   │
  │    ├─ YES → Bisection (safest) or Regula Falsi      │
  │    └─ NO  → Newton or Secant (faster)               │
  │                                                      │
  │  HYBRID APPROACH (recommended in practice):          │
  │    1. Start with Bisection to get near root          │
  │    2. Switch to Newton/Secant for fast convergence   │
  └─────────────────────────────────────────────────────┘
```

---

## 7. Rate of Convergence Proofs (Key Results)

### Newton's Method: Quadratic Convergence

From Taylor expansion about $r$:

$$0 = f(r) = f(x_n) + f'(x_n)(r - x_n) + \frac{f''(\xi)}{2}(r - x_n)^2$$

$$0 = f(x_n) + f'(x_n)(r - x_n) + \frac{f''(\xi)}{2}e_n^2$$

From Newton's formula: $f(x_n) = -f'(x_n)(x_{n+1} - x_n) = f'(x_n)(e_{n+1} - e_n)$

Substituting:

$$e_{n+1} = \frac{f''(\xi)}{2f'(x_n)} e_n^2$$

Therefore $p = 2$ with asymptotic error constant $C = \frac{|f''(r)|}{2|f'(r)|}$.

### Bisection: Linear Convergence

$$e_{n+1} \leq \frac{1}{2} e_n \implies p = 1, \; C = \frac{1}{2}$$

---

## 8. Stopping Criteria Comparison

| Criterion | Formula | Best For |
|-----------|---------|----------|
| Absolute | $\|x_{n+1} - x_n\| < \epsilon_a$ | When scale of root is known |
| Relative | $\frac{\|x_{n+1} - x_n\|}{\|x_{n+1}\|} < \epsilon_r$ | General purpose |
| Residual | $\|f(x_n)\| < \epsilon_f$ | When $f$ tolerance matters |
| Combined | All of above | Robust implementation |
| Max iterations | $n < N_{\max}$ | Safety net |

**Warning**: Small $|f(x)|$ does NOT guarantee small $|x - r|$ when $f'$ is small!

```
  Steep function:                Flat function:
  f(x) │    ╱                   f(x) │
       │   ╱                        │  ╱──────────
       │  ╱                         │ ╱
  ─────┼─╳────── x                 ─┼╱────╳───── x
       │╱                          │      
       ╱                            │
  small f → small error           small f → LARGE error!
```

---

## 9. Real-World Application: Hybrid Methods

In practice, production-quality root finders use hybrid strategies:

| Software | Method |
|----------|--------|
| MATLAB `fzero` | Brent's method (bisection + secant + inverse quadratic) |
| NumPy `scipy.optimize.brentq` | Brent's method |
| C++ Boost | TOMS 748 (guaranteed + superlinear) |

**Brent's Method**: Combines the safety of bisection with the speed of superlinear methods. If the fast method makes good progress, use it; otherwise, fall back to bisection.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Order of convergence $p$ | $e_{n+1} \approx C \cdot e_n^p$ |
| Efficiency index | $EI = p^{1/k}$; secant > Newton per evaluation |
| Linear convergence | ~3.3 iterations per digit |
| Quadratic convergence | Digits double each iteration |
| Multiple roots | Degrade convergence; use modified methods |
| Hybrid methods | Best practical choice (Brent's) |
| Stopping criteria | Use combined: absolute + relative + residual |

---

## Quick Revision Questions

1. **Define order of convergence and asymptotic error constant.**

2. **Rank bisection, Regula Falsi, secant, and Newton-Raphson by (a) convergence order, (b) efficiency index.**

3. **A method gives errors $0.1, 0.01, 0.001, 0.0001$. What is the order of convergence? Another method gives $0.1, 0.01, 0.0001, 10^{-8}$. What is its order?**

4. **Why does Newton's method slow down for multiple roots? State the modified formula.**

5. **Explain Brent's method conceptually. Why is it preferred in practice?**

6. **Design a selection strategy: given $f(x) = 0$ with no derivative and an interval $[a, b]$ where $f(a)f(b) < 0$, which method(s) would you use and why?**

---

[← Previous: Fixed-Point Iteration](05-fixed-point-iteration.md) | [Next: Unit 3 — Gaussian Elimination →](../03-System-of-Equations/01-gaussian-elimination.md)
