# Chapter 5.5: Characteristic Equation — Advanced Topics

[← Previous: Constant Coefficient Homogeneous](04-constant-coefficient-homogeneous.md) | [Back to README](../README.md) | [Next: Undetermined Coefficients →](../06-Non-Homogeneous-Linear-ODEs/01-undetermined-coefficients.md)

---

## Overview

This chapter explores advanced techniques related to the characteristic equation: factoring strategies, the Euler-Cauchy (equidimensional) equation that reduces to a constant-coefficient problem, and the D-operator formalism for compact notation.

---

## 1. Factoring the Characteristic Equation

### Common Factoring Strategies

| Equation Type | Strategy | Example |
|--------------|----------|---------|
| Quadratic | Quadratic formula | $m^2 - 5m + 6 = (m-2)(m-3)$ |
| Cubic with one obvious root | Factor out $(m - r)$, then quadratic | $m^3 - 6m^2 + 11m - 6$ |
| Biquadratic ($m^4$ with even powers) | Substitute $u = m^2$ | $m^4 + 5m^2 + 4 = 0$ |
| Symmetric coefficients | Test $m = 1$ or $m = -1$ | $m^3 - m^2 - m + 1$ |

### Example: Cubic Characteristic Equation

$y''' - 6y'' + 11y' - 6y = 0$

$m^3 - 6m^2 + 11m - 6 = 0$

Try $m = 1$: $1 - 6 + 11 - 6 = 0$ ✓

Synthetic division:

```
  1 │ 1  -6   11  -6
    │    1   -5    6
    ├─────────────────
    │ 1  -5    6   0
```

$m^3 - 6m^2 + 11m - 6 = (m-1)(m^2 - 5m + 6) = (m-1)(m-2)(m-3)$

$$y = c_1 e^x + c_2 e^{2x} + c_3 e^{3x}$$

---

## 2. The Euler-Cauchy Equation

### Standard Form

$$\boxed{x^n y^{(n)} + a_{n-1}x^{n-1}y^{(n-1)} + \cdots + a_1 xy' + a_0 y = 0}$$

Each term has the pattern: coefficient × $x^k y^{(k)}$.

### Second-Order Case

$$x^2 y'' + axy' + by = 0$$

### Trial Solution: $y = x^m$

$$y = x^m \implies y' = mx^{m-1} \implies y'' = m(m-1)x^{m-2}$$

Substituting:

$$x^2 \cdot m(m-1)x^{m-2} + ax \cdot mx^{m-1} + bx^m = 0$$

$$x^m[m(m-1) + am + b] = 0$$

### Indicial (Characteristic) Equation

$$\boxed{m(m-1) + am + b = 0 \implies m^2 + (a-1)m + b = 0}$$

### Three Cases (Paralleling Constant Coefficients)

```
┌──────────────────────────────────────────────────────┐
│   EULER-CAUCHY: x²y'' + axy' + by = 0                │
│   Indicial equation: m² + (a-1)m + b = 0             │
├──────────────────────────────────────────────────────┤
│                                                      │
│ Case 1: Distinct real roots m₁, m₂                   │
│         y = c₁x^m₁ + c₂x^m₂                         │
│                                                      │
│ Case 2: Repeated root m₁ = m₂ = m                    │
│         y = (c₁ + c₂ ln x)x^m                        │
│                                                      │
│ Case 3: Complex roots α ± iβ                         │
│         y = x^α[c₁cos(β ln x) + c₂sin(β ln x)]     │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 3. Euler-Cauchy — Worked Examples

### Example 1: Distinct Roots — $x^2y'' - 2xy' - 4y = 0$

Indicial equation: $m(m-1) - 2m - 4 = 0 \implies m^2 - 3m - 4 = 0$

$(m-4)(m+1) = 0 \implies m = 4, -1$

$$\boxed{y = c_1 x^4 + c_2 x^{-1}}$$

### Example 2: Repeated Root — $x^2y'' + xy' + y = 0$ (actually complex, see below)

$m(m-1) + m + 1 = 0 \implies m^2 + 1 = 0 \implies m = \pm i$

$\alpha = 0$, $\beta = 1$

$$\boxed{y = c_1\cos(\ln x) + c_2\sin(\ln x)}$$

### Example 3: Repeated Root — $x^2y'' - 3xy' + 4y = 0$

$m(m-1) - 3m + 4 = 0 \implies m^2 - 4m + 4 = 0 \implies (m-2)^2 = 0$

$m = 2$ (repeated)

$$\boxed{y = (c_1 + c_2 \ln x)x^2}$$

### Example 4: Complex Roots — $x^2y'' + xy' + 4y = 0$

$m(m-1) + m + 4 = 0 \implies m^2 + 4 = 0 \implies m = \pm 2i$

$\alpha = 0$, $\beta = 2$

$$\boxed{y = c_1\cos(2\ln x) + c_2\sin(2\ln x)}$$

---

## 4. Transformation to Constant Coefficients

The substitution $x = e^t$ (i.e., $t = \ln x$) transforms Euler-Cauchy into constant coefficients:

$$xy' = \frac{dy}{dt}, \quad x^2y'' = \frac{d^2y}{dt^2} - \frac{dy}{dt}$$

### Example: $x^2y'' + 3xy' + y = 0$

Let $x = e^t$, $D = d/dt$:

$$[D(D-1) + 3D + 1]y = 0 \implies (D^2 + 2D + 1)y = 0$$

$$(D+1)^2 y = 0 \implies y = (c_1 + c_2 t)e^{-t}$$

Back-substitute $t = \ln x$:

$$y = (c_1 + c_2 \ln x)\frac{1}{x}$$

---

## 5. The D-Operator Notation

### Definition

Let $D = \dfrac{d}{dx}$. Then:

$$D^n y = y^{(n)}, \quad D^2 y = y'', \quad \text{etc.}$$

### Operator Polynomial

$$L = a_n D^n + a_{n-1}D^{n-1} + \cdots + a_1 D + a_0$$

The ODE $L[y] = 0$ becomes $f(D)y = 0$ where $f(m) = 0$ is the characteristic equation.

### Useful Properties

| Property | Formula |
|----------|---------|
| $D(e^{ax}) = $ | $ae^{ax}$ |
| $f(D)e^{ax} = $ | $f(a)e^{ax}$ |
| $D^n(x^k) = $ | $k(k-1)\cdots(k-n+1)x^{k-n}$ |
| $(D - a)(D - b) = $ | $D^2 - (a+b)D + ab$ |
| Factoring | $f(D)$ factors like the polynomial $f(m)$ |

### Example with D-operator

$y'' - 3y' + 2y = 0$ becomes $(D^2 - 3D + 2)y = 0$

Factor: $(D-1)(D-2)y = 0$

This means either $(D-1)y = 0$ (giving $y = e^x$) or $(D-2)y = 0$ (giving $y = e^{2x}$).

---

## 6. Initial Value Problems — Complete Examples

### Example: IVP with Distinct Roots

$y'' - y = 0$, $y(0) = 3$, $y'(0) = -1$

$m^2 - 1 = 0 \implies m = \pm 1$

$y = c_1 e^x + c_2 e^{-x}$

$y(0) = c_1 + c_2 = 3$
$y'(0) = c_1 - c_2 = -1$

$c_1 = 1$, $c_2 = 2$

$$y = e^x + 2e^{-x}$$

### Example: IVP with Complex Roots

$y'' + 4y' + 8y = 0$, $y(0) = 1$, $y'(0) = 0$

$m = \dfrac{-4 \pm \sqrt{16-32}}{2} = -2 \pm 2i$

$y = e^{-2x}(c_1\cos 2x + c_2\sin 2x)$

$y(0) = c_1 = 1$

$y' = e^{-2x}[(-2c_1 + 2c_2)\cos 2x + (-2c_2 - 2c_1)\sin 2x]$

$y'(0) = -2c_1 + 2c_2 = 0 \implies c_2 = 1$

$$y = e^{-2x}(\cos 2x + \sin 2x)$$

---

## Summary Table

| Topic | Key Formula |
|-------|-------------|
| **Characteristic eq.** | $a_n m^n + \cdots + a_0 = 0$ |
| **Euler-Cauchy** | $x^n y^{(n)} + \cdots + a_0 y = 0$ |
| **Euler-Cauchy trial** | $y = x^m$ |
| **Euler-Cauchy repeated** | $(c_1 + c_2\ln x)x^m$ |
| **Euler-Cauchy complex** | $x^\alpha[c_1\cos(\beta\ln x) + c_2\sin(\beta\ln x)]$ |
| **Transformation** | $x = e^t$ converts Euler-Cauchy → constant coefficients |
| **D-operator** | $D = d/dx$; ODE becomes $f(D)y = 0$ |

---

## Quick Revision Questions

1. Solve the Euler-Cauchy equation $x^2y'' - xy' + y = 0$.

2. Factor $m^4 - 1 = 0$ and write the general solution of $y^{(4)} - y = 0$.

3. Transform $x^2y'' + 5xy' + 4y = 0$ using $x = e^t$ and solve.

4. Use the D-operator to show that $(D-a)(D-b)y = 0$ has solution $y = c_1e^{ax} + c_2 e^{bx}$.

5. Solve the third-order Euler-Cauchy equation $x^3y''' + 2x^2y'' - xy' + y = 0$ using $y = x^m$.

6. Solve the IVP: $y'' + y = 0$, $y(\pi/2) = 0$, $y'(\pi/2) = 1$.

---

[← Previous: Constant Coefficient Homogeneous](04-constant-coefficient-homogeneous.md) | [Back to README](../README.md) | [Next: Undetermined Coefficients →](../06-Non-Homogeneous-Linear-ODEs/01-undetermined-coefficients.md)
