# Chapter 5.4: Constant Coefficient Homogeneous ODEs

[← Previous: Reduction of Order](03-reduction-of-order.md) | [Back to README](../README.md) | [Next: Characteristic Equation →](05-characteristic-equation.md)

---

## Overview

When the coefficients of a linear homogeneous ODE are constants, the characteristic (auxiliary) equation technique provides a systematic, algebraic method for finding the general solution. This is one of the most important and frequently used methods in ODEs.

---

## 1. The Standard Form

$$\boxed{a_n y^{(n)} + a_{n-1} y^{(n-1)} + \cdots + a_1 y' + a_0 y = 0}$$

where $a_0, a_1, \ldots, a_n$ are **real constants** and $a_n \neq 0$.

### Key Insight

Trial solution: $y = e^{mx}$

$$y' = me^{mx}, \quad y'' = m^2 e^{mx}, \quad \ldots, \quad y^{(n)} = m^n e^{mx}$$

Substituting:

$$e^{mx}(a_n m^n + a_{n-1} m^{n-1} + \cdots + a_1 m + a_0) = 0$$

Since $e^{mx} \neq 0$:

$$\boxed{a_n m^n + a_{n-1} m^{n-1} + \cdots + a_1 m + a_0 = 0}$$

This is the **characteristic equation** (or **auxiliary equation**).

---

## 2. Three Cases for Second-Order ODEs

### $ay'' + by' + cy = 0$ with characteristic equation $am^2 + bm + c = 0$

$$m = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}$$

The discriminant $\Delta = b^2 - 4ac$ determines the nature of roots:

```
                    Discriminant Δ = b² - 4ac
                           │
              ┌────────────┼────────────┐
              │            │            │
           Δ > 0        Δ = 0        Δ < 0
              │            │            │
     Two distinct     Repeated     Complex conjugate
     real roots       real root       roots
     m₁ ≠ m₂          m₁ = m₂      m = α ± iβ
              │            │            │
              ▼            ▼            ▼
     y = c₁e^(m₁x)  y = (c₁+c₂x)e^(mx)  y = e^(αx)(c₁cosβx
       + c₂e^(m₂x)                          + c₂sinβx)
```

---

## 3. Case 1: Distinct Real Roots ($\Delta > 0$)

Roots: $m_1 \neq m_2$ (both real)

$$\boxed{y = c_1 e^{m_1 x} + c_2 e^{m_2 x}}$$

### Example 1: $y'' - 5y' + 6y = 0$

Characteristic equation: $m^2 - 5m + 6 = 0 \implies (m-2)(m-3) = 0$

$m_1 = 2$, $m_2 = 3$

$$y = c_1 e^{2x} + c_2 e^{3x}$$

### Example 2: $y'' + y' - 6y = 0$

$m^2 + m - 6 = 0 \implies (m+3)(m-2) = 0$

$m = -3, 2$

$$y = c_1 e^{-3x} + c_2 e^{2x}$$

---

## 4. Case 2: Repeated Real Roots ($\Delta = 0$)

Root: $m_1 = m_2 = m = -b/(2a)$

The two solutions are $e^{mx}$ and $xe^{mx}$:

$$\boxed{y = (c_1 + c_2 x)e^{mx}}$$

### Why $xe^{mx}$?

If we try $y = c_1 e^{mx} + c_2 e^{mx} = (c_1 + c_2)e^{mx}$, we get only **one** independent solution.

Reduction of order (with $y_1 = e^{mx}$, $P = b/a$, and $m = -b/(2a)$):

$$y_2 = e^{mx}\int \frac{e^{-(b/a)x}}{e^{2mx}}\,dx = e^{mx}\int \frac{e^{-(b/a)x}}{e^{-bx/a}}\,dx = e^{mx}\int dx = xe^{mx}$$

### Example 3: $y'' - 4y' + 4y = 0$

$m^2 - 4m + 4 = 0 \implies (m-2)^2 = 0$

$m = 2$ (repeated)

$$y = (c_1 + c_2 x)e^{2x}$$

### Example 4: $y'' + 6y' + 9y = 0$

$m^2 + 6m + 9 = 0 \implies (m + 3)^2 = 0$

$m = -3$ (repeated)

$$y = (c_1 + c_2 x)e^{-3x}$$

---

## 5. Case 3: Complex Conjugate Roots ($\Delta < 0$)

Roots: $m = \alpha \pm i\beta$ where $\alpha = -b/(2a)$, $\beta = \sqrt{4ac - b^2}/(2a)$

Using Euler's formula: $e^{i\beta x} = \cos\beta x + i\sin\beta x$

$$\boxed{y = e^{\alpha x}(c_1 \cos\beta x + c_2 \sin\beta x)}$$

Or equivalently: $y = Ae^{\alpha x}\cos(\beta x - \phi)$ (amplitude-phase form)

### Example 5: $y'' + 4y = 0$

$m^2 + 4 = 0 \implies m = \pm 2i$

$\alpha = 0$, $\beta = 2$

$$y = c_1 \cos 2x + c_2 \sin 2x$$

### Example 6: $y'' - 2y' + 5y = 0$

$m^2 - 2m + 5 = 0 \implies m = \frac{2 \pm \sqrt{4-20}}{2} = 1 \pm 2i$

$\alpha = 1$, $\beta = 2$

$$y = e^x(c_1 \cos 2x + c_2 \sin 2x)$$

### Solution Behavior

```
  CASE 3 SOLUTIONS

  α < 0: Damped oscillation       α = 0: Pure oscillation
  y │                               y │
    │  ╱╲                             │  ╱╲    ╱╲    ╱╲
    │ ╱  ╲   ╱╲                       │ ╱  ╲  ╱  ╲  ╱  ╲
  ──┼╱────╲─╱──╲──╱╲── x           ──┼╱────╲╱────╲╱────╲── x
    │      ╲    ╲╱  ╲╱                │      ╲    ╲╱    ╲╱
    │       ╲╱                        │
    │  envelope: e^(αx)               │  constant amplitude

  α > 0: Growing oscillation
  y │           ╱╲
    │      ╱╲  ╱  ╲
    │  ╱╲ ╱  ╲╱    ╲
  ──┼╱──╲╱─────────╲── x
    │    ╲           ╲
    │     ╲
    │  envelope: e^(αx)
```

---

## 6. Higher-Order Cases

### Third-Order: $a_3 m^3 + a_2 m^2 + a_1 m + a_0 = 0$

| Roots | General Solution |
|-------|------------------|
| $m_1, m_2, m_3$ distinct real | $c_1 e^{m_1 x} + c_2 e^{m_2 x} + c_3 e^{m_3 x}$ |
| $m_1 = m_2 \neq m_3$ | $(c_1 + c_2 x)e^{m_1 x} + c_3 e^{m_3 x}$ |
| $m_1 = m_2 = m_3 = m$ | $(c_1 + c_2 x + c_3 x^2)e^{mx}$ |
| $m_1$ real, $m_{2,3} = \alpha \pm i\beta$ | $c_1 e^{m_1 x} + e^{\alpha x}(c_2 \cos\beta x + c_3 \sin\beta x)$ |

### General Rule for Repeated Roots

If root $m$ has multiplicity $k$:

$$\text{Contribution: } (c_1 + c_2 x + c_3 x^2 + \cdots + c_k x^{k-1})e^{mx}$$

If complex root $\alpha + i\beta$ has multiplicity $k$:

$$\text{Contribution: } e^{\alpha x}\left[\sum_{j=0}^{k-1}(a_j \cos\beta x + b_j \sin\beta x)x^j\right]$$

---

## 7. Comprehensive Example

### $y^{(4)} - 2y''' + 2y'' - 2y' + y = 0$

Characteristic equation: $m^4 - 2m^3 + 2m^2 - 2m + 1 = 0$

Factor: $(m^2 + 1)(m^2 - 2m + 1) = 0$

$(m^2 + 1)(m-1)^2 = 0$

Roots: $m = 1$ (repeated), $m = \pm i$

$$\boxed{y = (c_1 + c_2 x)e^x + c_3 \cos x + c_4 \sin x}$$

---

## Summary Table

| Discriminant | Root Type | General Solution |
|:---:|---|---|
| $\Delta > 0$ | Distinct real: $m_1, m_2$ | $c_1 e^{m_1 x} + c_2 e^{m_2 x}$ |
| $\Delta = 0$ | Repeated: $m$ | $(c_1 + c_2 x)e^{mx}$ |
| $\Delta < 0$ | Complex: $\alpha \pm i\beta$ | $e^{\alpha x}(c_1\cos\beta x + c_2\sin\beta x)$ |
| — | Root $m$, multiplicity $k$ | $(c_1 + c_2 x + \cdots + c_k x^{k-1})e^{mx}$ |

---

## Quick Revision Questions

1. Solve $y'' - y = 0$.

2. Solve $y'' + 2y' + y = 0$.

3. Solve $y'' + 9y = 0$ and write the solution in amplitude-phase form.

4. Find the general solution of $y''' - y' = 0$.

5. Solve $y^{(4)} + 2y'' + y = 0$. *(Hint: $(m^2+1)^2 = 0$)*

6. For $y'' - 4y' + 13y = 0$, classify the motion as damped, undamped, or growing oscillation.

---

[← Previous: Reduction of Order](03-reduction-of-order.md) | [Back to README](../README.md) | [Next: Characteristic Equation →](05-characteristic-equation.md)
