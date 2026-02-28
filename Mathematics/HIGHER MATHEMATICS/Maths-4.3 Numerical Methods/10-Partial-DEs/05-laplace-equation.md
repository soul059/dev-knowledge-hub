# Chapter 10.5: Numerical Solution of the Laplace Equation

[← Previous: Wave Equation](04-wave-equation.md) | [Back to Main README →](../README.md)

---

## Overview

The **Laplace equation** $\nabla^2 u = 0$ and the **Poisson equation** $\nabla^2 u = f$ are prototype elliptic PDEs describing steady-state phenomena. Since there is no time variable, the solution is found by solving a **large system of simultaneous equations** — typically using **iterative methods**.

---

## 1. Problem Statement

$$\frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2} = 0 \quad \text{(Laplace)}$$

$$\frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2} = f(x,y) \quad \text{(Poisson)}$$

on a rectangular domain with Dirichlet boundary conditions $u = g$ on all boundaries.

---

## 2. Five-Point Finite Difference Scheme

Using the standard central difference with $h = \Delta x = \Delta y$:

$$\boxed{u_{i+1,j} + u_{i-1,j} + u_{i,j+1} + u_{i,j-1} - 4u_{i,j} = 0}$$

This says: **the value at each interior point equals the average of its four neighbours**.

$$\boxed{u_{i,j} = \frac{1}{4}(u_{i+1,j} + u_{i-1,j} + u_{i,j+1} + u_{i,j-1})}$$

For Poisson: $u_{i,j} = \frac{1}{4}(u_{i+1,j} + u_{i-1,j} + u_{i,j+1} + u_{i,j-1} - h^2 f_{i,j})$

---

## 3. System of Equations

For an $(M-1) \times (N-1)$ interior grid, we get $(M-1)(N-1)$ equations in $(M-1)(N-1)$ unknowns.

The system $A\mathbf{u} = \mathbf{b}$ has a **block tridiagonal** structure:

$$A = \begin{bmatrix} T & I & & \\ I & T & I & \\ & \ddots & \ddots & \ddots \\ & & I & T \end{bmatrix}, \quad T = \begin{bmatrix} -4 & 1 & & \\ 1 & -4 & 1 & \\ & \ddots & \ddots & \ddots \\ & & 1 & -4 \end{bmatrix}$$

Direct solve: $O(M^3)$ — impractical for large grids. Use **iterative methods**.

---

## 4. Jacobi Iteration

Update all points simultaneously using values from the previous iteration:

$$\boxed{u_{i,j}^{(k+1)} = \frac{1}{4}\left(u_{i+1,j}^{(k)} + u_{i-1,j}^{(k)} + u_{i,j+1}^{(k)} + u_{i,j-1}^{(k)}\right)}$$

**Convergence rate:** spectral radius $\rho = \cos(\pi h)$, slow for fine grids.

---

## 5. Gauss–Seidel Iteration

Use updated values as soon as they are available:

$$\boxed{u_{i,j}^{(k+1)} = \frac{1}{4}\left(u_{i+1,j}^{(k)} + u_{i-1,j}^{(k+1)} + u_{i,j+1}^{(k)} + u_{i,j-1}^{(k+1)}\right)}$$

Converges **twice as fast** as Jacobi: $\rho_{\text{GS}} = \rho_{\text{J}}^2 = \cos^2(\pi h)$.

---

## 6. SOR (Successive Over-Relaxation)

$$\boxed{u_{i,j}^{(k+1)} = (1-\omega)u_{i,j}^{(k)} + \frac{\omega}{4}\left(u_{i+1,j}^{(k)} + u_{i-1,j}^{(k+1)} + u_{i,j+1}^{(k)} + u_{i,j-1}^{(k+1)}\right)}$$

**Optimal relaxation parameter** for a square grid:

$$\boxed{\omega_{\text{opt}} = \frac{2}{1 + \sin(\pi h)} \approx \frac{2}{1+\pi h}}$$

| Method | Iterations to converge ($\varepsilon$) | Rate |
|--------|---------------------------------------|------|
| Jacobi | $O(h^{-2})$ | Slow |
| Gauss–Seidel | $O(h^{-2})$ (half of Jacobi) | Moderate |
| SOR ($\omega_{\text{opt}}$) | $O(h^{-1})$ | **Fast** |

---

## 7. Worked Example

Solve $\nabla^2 u = 0$ on the unit square with:
- $u = 0$ on bottom, left, right
- $u = 100$ on top

Take $h = 1/3$ (interior grid: $2 \times 2 = 4$ unknowns).

```
    100    100    100
  ┌──────┬──────┬──────┐
  │      │      │      │
0 │  u₁  │  u₂  │      │ 0
  │      │      │      │
  ├──────┼──────┤      │
  │      │      │      │
0 │  u₃  │  u₄  │      │ 0
  │      │      │      │
  └──────┴──────┴──────┘
     0      0      0
```

**Equations (five-point stencil):**

$$u_1: \quad 0 + u_2 + 100 + u_3 - 4u_1 = 0$$
$$u_2: \quad u_1 + 0 + 100 + u_4 - 4u_2 = 0$$
$$u_3: \quad 0 + u_4 + u_1 + 0 - 4u_3 = 0$$
$$u_4: \quad u_3 + 0 + u_2 + 0 - 4u_4 = 0$$

System:

$$\begin{bmatrix} -4 & 1 & 1 & 0 \\ 1 & -4 & 0 & 1 \\ 1 & 0 & -4 & 1 \\ 0 & 1 & 1 & -4 \end{bmatrix} \begin{bmatrix} u_1 \\ u_2 \\ u_3 \\ u_4 \end{bmatrix} = \begin{bmatrix} -100 \\ -100 \\ 0 \\ 0 \end{bmatrix}$$

By symmetry $u_1 = u_2$ and $u_3 = u_4$. Substituting:

$$-4u_1 + u_1 + 100 + u_3 = 0 \implies -3u_1 + u_3 = -100$$
$$-4u_3 + u_3 + u_1 = 0 \implies u_1 - 3u_3 = 0 \implies u_1 = 3u_3$$

$$-3(3u_3) + u_3 = -100 \implies -8u_3 = -100 \implies u_3 = 12.5$$

$$u_1 = 37.5$$

$$\boxed{u_1 = u_2 = 37.5, \quad u_3 = u_4 = 12.5}$$

### Gauss–Seidel Verification (starting from $u=0$)

| Iter | $u_1$ | $u_2$ | $u_3$ | $u_4$ |
|------|-------|-------|-------|-------|
| 0 | 0 | 0 | 0 | 0 |
| 1 | 25.00 | 31.25 | 6.25 | 9.38 |
| 2 | 35.16 | 36.13 | 11.32 | 11.86 |
| 3 | 37.00 | 37.21 | 12.30 | 12.38 |
| 4 | 37.40 | 37.44 | 12.46 | 12.48 |
| ... | 37.50 | 37.50 | 12.50 | 12.50 |

Converges to the exact solution within ~8 iterations.

---

## 8. ADI Method (Alternating Direction Implicit)

For large problems, **ADI** splits each iteration into two half-steps:

**Step 1** (implicit in $x$): $-u_{i-1,j}^* + (2+h^2\alpha)u_{i,j}^* - u_{i+1,j}^* = u_{i,j-1}^n + (−2+h^2\alpha)u_{i,j}^n + u_{i,j+1}^n$

**Step 2** (implicit in $y$): similar with roles of $x$ and $y$ swapped.

Each half-step requires solving a **tridiagonal** system — very efficient!

---

## Summary Table

| Method | Formula | Convergence | Notes |
|--------|---------|-------------|-------|
| **5-point stencil** | $u_{i,j} = \frac{1}{4}\sum\text{neighbours}$ | — | $O(h^2)$ accuracy |
| **Jacobi** | All old values | $\rho = \cos\pi h$ | Simple, parallelisable |
| **Gauss–Seidel** | Use new values immediately | $\rho^2 = \cos^2\pi h$ | 2× faster than Jacobi |
| **SOR** | Over-relaxation factor $\omega$ | $O(h^{-1})$ with $\omega_\text{opt}$ | Best iterative |
| **ADI** | Alternating direction | $O(h^{-1})$ | Tridiagonal solves |
| **Direct (LU)** | Band solver | Exact (1 step) | $O(M^3)$ memory |

---

## Quick Revision Questions

1. **Write the five-point FD equation for $\nabla^2 u = 0$. What does it mean physically?**

2. **Solve $\nabla^2 u = 0$ on a square with $u=0$ on three sides and $u=50$ on top, using $h=1/2$ (one interior point).**

3. **Compare Jacobi, Gauss–Seidel, and SOR for solving the Laplace equation.**

4. **What is the optimal SOR parameter for a $10 \times 10$ grid ($h = 1/11$)?**

5. **Write the Poisson equation FD scheme with source term $f(x,y)$.**

6. **Describe the ADI method and why it is efficient.**

---

[← Previous: Wave Equation](04-wave-equation.md) | [Back to Main README →](../README.md)
