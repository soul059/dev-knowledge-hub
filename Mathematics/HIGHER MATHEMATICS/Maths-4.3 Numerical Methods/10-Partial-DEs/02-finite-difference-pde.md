# Chapter 10.2: Finite Difference Methods for PDEs

[← Previous: Classification of PDEs](01-classification.md) | [Next: Heat Equation →](03-heat-equation.md)

---

## Overview

Finite difference methods replace partial derivatives with algebraic approximations on a **grid**. This chapter covers grid construction, difference formulae for partial derivatives, and the fundamental concepts of consistency, stability, and convergence.

---

## 1. Grid Construction

Discretise the domain with step sizes $h = \Delta x$, $k = \Delta y$ (or $\Delta t$):

$$x_i = x_0 + ih, \quad y_j = y_0 + jk$$

```
  y_j  ──●──────●──────●──────●──────●──
        │      │      │      │      │
  y_{j} ──●──────●──────●──────●──────●──
        │      │      │      │      │
  y_{j-1}──●──────●──────●──────●──────●──
        │      │      │      │      │
       x_{i-1} x_i  x_{i+1}x_{i+2}
             h     h     h
```

Notation: $u_{i,j} \approx u(x_i, y_j)$

---

## 2. Finite Difference Approximations

### Second-order central differences

$$\frac{\partial u}{\partial x}\bigg|_{i,j} \approx \frac{u_{i+1,j} - u_{i-1,j}}{2h} + O(h^2)$$

$$\frac{\partial^2 u}{\partial x^2}\bigg|_{i,j} \approx \frac{u_{i+1,j} - 2u_{i,j} + u_{i-1,j}}{h^2} + O(h^2)$$

$$\frac{\partial^2 u}{\partial y^2}\bigg|_{i,j} \approx \frac{u_{i,j+1} - 2u_{i,j} + u_{i,j-1}}{k^2} + O(k^2)$$

$$\frac{\partial^2 u}{\partial x \partial y}\bigg|_{i,j} \approx \frac{u_{i+1,j+1} - u_{i+1,j-1} - u_{i-1,j+1} + u_{i-1,j-1}}{4hk} + O(h^2 + k^2)$$

### Forward difference in time

$$\frac{\partial u}{\partial t}\bigg|_{i,n} \approx \frac{u_i^{n+1} - u_i^n}{k} + O(k)$$

---

## 3. The Five-Point Stencil (Laplacian)

For $\nabla^2 u = u_{xx} + u_{yy}$ with $h = k$:

$$\boxed{\nabla^2 u \approx \frac{u_{i+1,j} + u_{i-1,j} + u_{i,j+1} + u_{i,j-1} - 4u_{i,j}}{h^2}}$$

```
              u_{i,j+1}
                 │
                 │
  u_{i-1,j} ── u_{i,j} ── u_{i+1,j}     Weights: [0,1,0]
                 │                                 [1,-4,1]
                 │                                 [0,1,0]
              u_{i,j-1}
```

This is $O(h^2)$ accurate.

---

## 4. Nine-Point Stencil (Higher Order)

For improved accuracy ($O(h^4)$ for Laplace):

$$\nabla^2 u \approx \frac{1}{6h^2}\begin{bmatrix} 1 & 4 & 1 \\ 4 & -20 & 4 \\ 1 & 4 & 1 \end{bmatrix}$$

---

## 5. Consistency, Stability, Convergence

### The Lax Equivalence Theorem

> For a **consistent** finite difference scheme applied to a well-posed linear PDE, **stability** $\iff$ **convergence**.

| Property | Definition |
|----------|-----------|
| **Consistency** | Truncation error → 0 as $h, k \to 0$ |
| **Stability** | Errors do not grow unboundedly in time |
| **Convergence** | Numerical solution → exact solution as $h, k \to 0$ |

```
  Consistency + Stability = Convergence
       │            │            │
       ▼            ▼            ▼
  Scheme approx.  Errors       Solution
  the PDE well    don't grow    is correct
```

---

## 6. Stability Analysis: Von Neumann Method

Substitute $u_j^n = \xi^n e^{i\beta jh}$ (Fourier mode) into the scheme and find the **amplification factor** $\xi$:

$$\text{Stability} \iff |\xi(\beta)| \leq 1 \quad \forall \beta$$

### Example: FTCS for Heat Equation

$u_i^{n+1} = u_i^n + r(u_{i+1}^n - 2u_i^n + u_{i-1}^n)$ where $r = \alpha k/h^2$

$$\xi = 1 - 4r\sin^2(\beta h/2)$$

Stability requires $|1 - 4r\sin^2(\beta h/2)| \leq 1$, giving:

$$\boxed{r = \frac{\alpha k}{h^2} \leq \frac{1}{2}}$$

---

## 7. Explicit vs Implicit Schemes

| Feature | Explicit | Implicit |
|---------|----------|----------|
| **Formula** | $u^{n+1}$ from $u^n$ directly | System of equations at each step |
| **Stability** | Conditional ($k \leq Ch^2$) | Often unconditional |
| **Cost per step** | Low (direct evaluation) | High (solve linear system) |
| **Accuracy** | Usually $O(k) + O(h^2)$ | Can achieve $O(k^2) + O(h^2)$ |
| **Example** | FTCS, Lax–Friedrichs | Backward Euler, Crank–Nicolson |

---

## 8. Boundary Condition Implementation

### Dirichlet ($u = g$ on boundary)

Simply set $u_{i,j} = g(x_i, y_j)$ at boundary nodes.

### Neumann ($\frac{\partial u}{\partial n} = g$ on boundary)

Use a **ghost point** outside the domain:

$$\frac{u_{1,j} - u_{-1,j}}{2h} = g_0 \implies u_{-1,j} = u_{1,j} - 2hg_0$$

Substitute into the interior equation to eliminate the ghost point.

---

## Summary Table

| Concept | Key Formula |
|---------|-------------|
| **$u_{xx}$ central** | $(u_{i+1}-2u_i+u_{i-1})/h^2$ |
| **5-point Laplacian** | $(u_{i+1,j}+u_{i-1,j}+u_{i,j+1}+u_{i,j-1}-4u_{i,j})/h^2$ |
| **Truncation error** | $O(h^2 + k^2)$ for central, $O(k)$ for forward-time |
| **Lax equivalence** | Consistency + Stability $\iff$ Convergence |
| **Von Neumann** | $|\xi(\beta)| \leq 1$ for all $\beta$ |
| **CFL condition** | $r = \alpha k/h^2 \leq 1/2$ (FTCS for heat) |

---

## Quick Revision Questions

1. **Write the 5-point FD stencil for the Laplacian $u_{xx} + u_{yy}$.**

2. **State the Lax Equivalence Theorem.**

3. **Perform Von Neumann stability analysis for the FTCS scheme for the heat equation.**

4. **How is the Neumann boundary condition $u_x(0,t) = 0$ implemented using ghost points?**

5. **Compare explicit and implicit FD schemes for time-dependent PDEs.**

6. **What is the order of accuracy of the central difference approximation $u_{xx} \approx (u_{i+1}-2u_i+u_{i-1})/h^2$?**

---

[← Previous: Classification of PDEs](01-classification.md) | [Next: Heat Equation →](03-heat-equation.md)
