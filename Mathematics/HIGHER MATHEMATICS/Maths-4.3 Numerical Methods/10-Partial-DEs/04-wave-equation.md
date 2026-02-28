# Chapter 10.4: Numerical Solution of the Wave Equation

[← Previous: Heat Equation](03-heat-equation.md) | [Next: Laplace Equation →](05-laplace-equation.md)

---

## Overview

The **wave equation** $u_{tt} = c^2 u_{xx}$ is the prototype hyperbolic PDE. Numerical methods must respect the **finite speed of propagation** and the **CFL condition**. The explicit central-difference scheme is the standard approach.

---

## 1. Problem Statement

$$\frac{\partial^2 u}{\partial t^2} = c^2 \frac{\partial^2 u}{\partial x^2}, \quad 0 < x < L, \quad t > 0$$

**Initial conditions:**
- $u(x,0) = f(x)$ (initial displacement)
- $u_t(x,0) = g(x)$ (initial velocity)

**Boundary conditions:** $u(0,t) = u(L,t) = 0$ (fixed ends)

Grid: $x_i = ih$, $t_n = nk$, $\lambda = \frac{ck}{h}$ (Courant number).

---

## 2. Explicit Scheme

Central differences in both space and time:

$$\frac{u_i^{n+1} - 2u_i^n + u_i^{n-1}}{k^2} = c^2 \frac{u_i^{n+1} - 2u_i^n + u_{i-1}^n}{h^2}$$

$$\boxed{u_i^{n+1} = 2(1-\lambda^2)u_i^n + \lambda^2(u_{i+1}^n + u_{i-1}^n) - u_i^{n-1}}$$

where $\lambda = ck/h$.

```
  n+1:            ● u_i^{n+1}    (computed)
                  │
  n:     ● ───── ● ───── ●       (known)
       u_{i-1}  u_i    u_{i+1}
                  │
  n-1:            ● u_i^{n-1}    (known)
```

**Three time levels** needed — requires special treatment of the first step.

---

## 3. Starting the Computation ($n=0 \to n=1$)

At $n=0$, we know $u^0 = f(x)$ but need both $u^0$ and $u^{-1}$ (fictitious level).

Using $u_t(x,0) = g(x)$:

$$u_i^{-1} \approx u_i^0 - kg(x_i) + \frac{k^2}{2}c^2 f''(x_i)$$

Substituting into the scheme for $n=0$:

$$\boxed{u_i^1 = (1-\lambda^2)f(x_i) + \frac{\lambda^2}{2}[f(x_{i+1})+f(x_{i-1})] + kg(x_i)}$$

For the common case $g(x) = 0$:

$$u_i^1 = (1-\lambda^2)f(x_i) + \frac{\lambda^2}{2}[f(x_{i+1})+f(x_{i-1})]$$

---

## 4. CFL Condition

**Stability requirement:**

$$\boxed{\lambda = \frac{ck}{h} \leq 1}$$

This is the **Courant–Friedrichs–Lewy (CFL) condition**.

**Physical interpretation:** The numerical domain of dependence must contain the physical domain of dependence.

```
  If λ ≤ 1 (CFL satisfied):        If λ > 1 (CFL violated):

       ● u_i^{n+1}                      ● u_i^{n+1}
      / \                              / \
     /   \  numerical                ╱     ╲  numerical
    /     \  domain                 ╱       ╲  domain
   ●───────●                      ●─────────●
   ╲physical╱                       ╲ phys╱
    ╲domain╱                         ╲ ╱
                                   Physical domain extends
   Numerical ⊇ Physical            OUTSIDE numerical
   ✅ Stable                       ❌ Unstable
```

**Special case** $\lambda = 1$: The scheme is **exact** (reproduces d'Alembert's solution).

---

## 5. Worked Example

Solve $u_{tt} = u_{xx}$ on $[0,1]$, with $u(x,0) = \sin(\pi x)$, $u_t(x,0) = 0$, $u(0,t)=u(1,t)=0$.

Take $h = 0.25$, $k = 0.25$, so $\lambda = 1$ (exact case).

**Exact solution:** $u(x,t) = \cos(\pi t)\sin(\pi x)$

### Step 0: Initial values $u^0$

| $i$ | $x_i$ | $u_i^0$ |
|-----|--------|---------|
| 0 | 0 | 0 |
| 1 | 0.25 | 0.7071 |
| 2 | 0.50 | 1.0000 |
| 3 | 0.75 | 0.7071 |
| 4 | 1.00 | 0 |

### Step 1: First time step ($n=0 \to 1$, $g=0$, $\lambda=1$)

$$u_i^1 = 0 \cdot f(x_i) + \frac{1}{2}[f(x_{i+1}) + f(x_{i-1})]$$

$$u_1^1 = \frac{1}{2}(0 + 1.0000) = 0.5000$$

$$u_2^1 = \frac{1}{2}(0.7071 + 0.7071) = 0.7071$$

$$u_3^1 = \frac{1}{2}(1.0000 + 0) = 0.5000$$

Exact at $t=0.25$: $\cos(0.25\pi)\sin(\pi x) = 0.7071 \sin(\pi x)$

| $x$ | Computed | Exact | Error |
|------|----------|-------|-------|
| 0.25 | 0.5000 | 0.5000 | 0 |
| 0.50 | 0.7071 | 0.7071 | 0 |
| 0.75 | 0.5000 | 0.5000 | 0 |

Exact (as predicted when $\lambda = 1$)!

### Step 2: $n=1 \to 2$

$$u_i^2 = 0 \cdot u_i^1 + 1 \cdot (u_{i+1}^1 + u_{i-1}^1) - u_i^0$$

$$u_1^2 = (0 + 0.7071) - 0.7071 = 0$$

$$u_2^2 = (0.5000 + 0.5000) - 1.0000 = 0$$

$$u_3^2 = (0.7071 + 0) - 0.7071 = 0$$

At $t = 0.5$: $u = \cos(0.5\pi)\sin(\pi x) = 0$ ✅

---

## 6. Higher-Dimensional Wave Equation

For $u_{tt} = c^2(u_{xx} + u_{yy})$ on a 2D grid:

$$u_{i,j}^{n+1} = 2u_{i,j}^n - u_{i,j}^{n-1} + \lambda_x^2(u_{i+1,j}^n - 2u_{i,j}^n + u_{i-1,j}^n) + \lambda_y^2(u_{i,j+1}^n - 2u_{i,j}^n + u_{i,j-1}^n)$$

CFL: $\lambda_x^2 + \lambda_y^2 \leq 1$ where $\lambda_x = ck/h_x$, $\lambda_y = ck/h_y$.

---

## 7. Implicit Methods for the Wave Equation

The **Newmark-$\beta$** family:

$$u^{n+1} = u^n + ku_t^n + k^2\left[(\tfrac{1}{2}-\beta)a^n + \beta a^{n+1}\right]$$

| $\beta$ | Method | Stability |
|---------|--------|-----------|
| 0 | Explicit central difference | $\lambda \leq 1$ |
| 1/4 | Average acceleration | Unconditional |
| 1/6 | Linear acceleration | Conditional |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Scheme** | $u_i^{n+1} = 2(1-\lambda^2)u_i^n + \lambda^2(u_{i+1}^n + u_{i-1}^n) - u_i^{n-1}$ |
| **Courant number** | $\lambda = ck/h$ |
| **CFL condition** | $\lambda \leq 1$ |
| **Accuracy** | $O(k^2) + O(h^2)$ |
| **Special: $\lambda=1$** | Exact (no numerical error) |
| **First step** | Special formula using initial velocity |
| **2D CFL** | $\lambda_x^2 + \lambda_y^2 \leq 1$ |

---

## Quick Revision Questions

1. **Write the explicit FD scheme for $u_{tt} = c^2 u_{xx}$. State the CFL condition.**

2. **Derive the formula for the first time step when $u_t(x,0) = 0$.**

3. **Why is $\lambda = 1$ special? What happens to the numerical solution?**

4. **Solve two time steps: $u_{tt} = 4u_{xx}$ on $[0,1]$, $u(x,0) = \sin(2\pi x)$, $u_t(x,0) = 0$, $h = 0.25$, $k = 0.125$.**

5. **Explain the CFL condition geometrically in terms of domains of dependence.**

6. **What is the CFL condition in 2D? How does it differ from 1D?**

---

[← Previous: Heat Equation](03-heat-equation.md) | [Next: Laplace Equation →](05-laplace-equation.md)
