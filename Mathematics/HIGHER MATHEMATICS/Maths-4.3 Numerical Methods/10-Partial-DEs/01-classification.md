# Chapter 10.1: Classification of Partial Differential Equations

[← Previous: Cubic Spline Fitting](../09-Curve-Fitting/04-cubic-spline-fitting.md) | [Next: Finite Difference Methods for PDEs →](02-finite-difference-pde.md)

---

## Overview

Partial Differential Equations (PDEs) involve functions of **multiple independent variables** and their partial derivatives. The **classification** of second-order linear PDEs into elliptic, parabolic, and hyperbolic types determines the nature of solutions and the appropriate numerical methods.

---

## 1. General Second-Order Linear PDE

$$\boxed{A\frac{\partial^2 u}{\partial x^2} + 2B\frac{\partial^2 u}{\partial x \partial y} + C\frac{\partial^2 u}{\partial y^2} + D\frac{\partial u}{\partial x} + E\frac{\partial u}{\partial y} + Fu = G}$$

where $A, B, C, D, E, F, G$ may be functions of $x, y$.

The **discriminant** $\Delta = B^2 - AC$ determines the type:

$$\boxed{\begin{cases} B^2 - AC < 0 & \Rightarrow \textbf{Elliptic} \\ B^2 - AC = 0 & \Rightarrow \textbf{Parabolic} \\ B^2 - AC > 0 & \Rightarrow \textbf{Hyperbolic} \end{cases}}$$

---

## 2. The Three Types

### Elliptic ($B^2 - AC < 0$)

| Property | Detail |
|----------|--------|
| **Prototype** | Laplace equation: $\nabla^2 u = u_{xx} + u_{yy} = 0$ |
| **Physics** | Steady-state heat, electrostatics, potential flow |
| **Boundary conditions** | Dirichlet or Neumann on closed boundary |
| **Solution character** | Smooth, no characteristics, maximum principle |

### Parabolic ($B^2 - AC = 0$)

| Property | Detail |
|----------|--------|
| **Prototype** | Heat equation: $u_t = \alpha u_{xx}$ |
| **Physics** | Heat conduction, diffusion |
| **Conditions** | Initial condition + boundary conditions |
| **Solution character** | Smoothing, information propagates at infinite speed |

### Hyperbolic ($B^2 - AC > 0$)

| Property | Detail |
|----------|--------|
| **Prototype** | Wave equation: $u_{tt} = c^2 u_{xx}$ |
| **Physics** | Vibrations, acoustics, electromagnetic waves |
| **Conditions** | Initial conditions ($u$ and $u_t$ at $t=0$) + boundary |
| **Solution character** | Propagating waves, finite speed, characteristics |

---

## 3. Visual Comparison

```
  ELLIPTIC                 PARABOLIC              HYPERBOLIC
  (Steady-state)           (Diffusion)            (Waves)

  ┌─────────────┐          t                      t
  │             │          │  ╲  diffusion         │   ╲  ╱  characteristics
  │  ∇²u = 0   │          │   ╲                   │    ╲╱
  │             │          │    ╲                  │    ╱╲
  │  Boundary   │          │─────────── x          │   ╱  ╲
  │  all around │          initial condition       │  ╱    ╲
  └─────────────┘                                  └──────── x
                                                   initial conditions
  No time variable         Marches in time         Marches in time
  BVP everywhere           IVP in time             IVP in time
```

---

## 4. Standard Examples

| PDE | $A$ | $B$ | $C$ | $B^2-AC$ | Type |
|-----|-----|-----|-----|----------|------|
| $u_{xx} + u_{yy} = 0$ | 1 | 0 | 1 | $-1$ | Elliptic |
| $u_t = u_{xx}$ ($y=t$) | 1 | 0 | 0 | $0$ | Parabolic |
| $u_{tt} = c^2 u_{xx}$ | $c^2$ | 0 | $-1$ | $c^2$ | Hyperbolic |
| $u_{xx} + 2u_{xy} + u_{yy} = 0$ | 1 | 1 | 1 | $0$ | Parabolic |
| $u_{xx} - u_{yy} = 0$ | 1 | 0 | $-1$ | $1$ | Hyperbolic |
| $y u_{xx} + u_{yy} = 0$ | $y$ | 0 | 1 | $-y$ | Depends on $y$ |

The last example is **Tricomi's equation** — elliptic for $y > 0$, hyperbolic for $y < 0$.

---

## 5. Boundary and Initial Conditions

| Type | Required Conditions | Domain |
|------|-------------------|--------|
| **Elliptic** | BCs on entire closed boundary | Closed region |
| **Parabolic** | IC at $t=0$ + BCs at boundaries in $x$ | Open in $t$ |
| **Hyperbolic** | ICs ($u, u_t$) at $t=0$ + BCs in $x$ | Open in $t$ |

**Condition types:**
- **Dirichlet**: $u = g$ on boundary
- **Neumann**: $\frac{\partial u}{\partial n} = g$ on boundary
- **Robin (mixed)**: $\alpha u + \beta \frac{\partial u}{\partial n} = g$

---

## 6. Well-Posedness (Hadamard)

A PDE problem is **well-posed** if:
1. A solution **exists**
2. The solution is **unique**
3. The solution **depends continuously** on the data

Matching the correct type of boundary/initial conditions to the PDE type ensures well-posedness.

---

## 7. Characteristics

Characteristic curves satisfy:

$$A \, dy^2 - 2B \, dx \, dy + C \, dx^2 = 0$$

| Type | Characteristics |
|------|----------------|
| Elliptic | No real characteristics (complex) |
| Parabolic | One family of real characteristics |
| Hyperbolic | Two families of real characteristics |

For the wave equation $u_{tt} = c^2 u_{xx}$: characteristics are $x \pm ct = \text{const}$ (the wave fronts).

---

## Summary Table

| Feature | Elliptic | Parabolic | Hyperbolic |
|---------|----------|-----------|------------|
| **Discriminant** | $B^2-AC < 0$ | $B^2-AC = 0$ | $B^2-AC > 0$ |
| **Prototype** | $u_{xx}+u_{yy}=0$ | $u_t = \alpha u_{xx}$ | $u_{tt}=c^2u_{xx}$ |
| **Physics** | Equilibrium | Diffusion | Wave propagation |
| **Time dependence** | Steady state | Unsteady | Unsteady |
| **Characteristics** | None (complex) | 1 family | 2 families |
| **Domain of dependence** | Entire domain | Past cone | Past light cone |
| **Numerical method** | Relaxation (Jacobi, SOR) | Explicit/implicit time-stepping | Explicit time-stepping |

---

## Quick Revision Questions

1. **Classify the PDE $3u_{xx} + 4u_{xy} + u_{yy} = 0$. Compute $B^2-AC$.**

2. **Name the prototype PDE for each type (elliptic, parabolic, hyperbolic).**

3. **What boundary/initial conditions are needed for the heat equation on $[0,L]$?**

4. **What are Hadamard's three conditions for well-posedness?**

5. **Find the characteristics of the wave equation $u_{tt} = 4u_{xx}$.**

6. **Why does Tricomi's equation $yu_{xx} + u_{yy} = 0$ change type?**

---

[← Previous: Cubic Spline Fitting](../09-Curve-Fitting/04-cubic-spline-fitting.md) | [Next: Finite Difference Methods for PDEs →](02-finite-difference-pde.md)
