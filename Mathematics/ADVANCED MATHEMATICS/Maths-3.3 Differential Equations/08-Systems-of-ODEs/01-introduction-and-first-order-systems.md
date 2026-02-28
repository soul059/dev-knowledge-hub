# Chapter 8.1: Introduction and Conversion to First-Order Systems

[← Previous: Beam Deflection](../07-Applications-Higher-Order/04-beam-deflection.md) | [Back to README](../README.md) | [Next: Matrix Method →](02-matrix-method.md)

---

## Overview

Many physical systems involve multiple interacting quantities, each governed by a differential equation. Such **systems of ODEs** naturally arise in population dynamics, coupled circuits, and mechanical vibrations. We also show that any single higher-order ODE can be written as a first-order system.

---

## 1. What is a System of ODEs?

A system of $n$ first-order ODEs:

$$\frac{dx_1}{dt} = f_1(t, x_1, x_2, \ldots, x_n)$$
$$\frac{dx_2}{dt} = f_2(t, x_1, x_2, \ldots, x_n)$$
$$\vdots$$
$$\frac{dx_n}{dt} = f_n(t, x_1, x_2, \ldots, x_n)$$

In vector form: $\mathbf{x}' = \mathbf{f}(t, \mathbf{x})$

### Linear Systems

$$\mathbf{x}' = A(t)\,\mathbf{x} + \mathbf{g}(t)$$

where $A(t)$ is an $n \times n$ matrix and $\mathbf{g}(t)$ is a vector.

If $A$ is constant and $\mathbf{g} = \mathbf{0}$: **constant-coefficient homogeneous** system.

---

## 2. Conversion: Higher-Order ODE → First-Order System

### Method

For an $n$-th order ODE $y^{(n)} = F(t, y, y', \ldots, y^{(n-1)})$:

**Introduce state variables**:
$$x_1 = y, \quad x_2 = y', \quad x_3 = y'', \quad \ldots, \quad x_n = y^{(n-1)}$$

Then:
$$x_1' = x_2$$
$$x_2' = x_3$$
$$\vdots$$
$$x_{n-1}' = x_n$$
$$x_n' = F(t, x_1, x_2, \ldots, x_n)$$

```
  CONVERSION FLOWCHART

  n-th order ODE
       │
       ▼
  Set x₁ = y, x₂ = y', ..., xₙ = y⁽ⁿ⁻¹⁾
       │
       ▼
  Write x₁' = x₂, x₂' = x₃, ...
       │
       ▼
  Express highest derivative from original ODE
       │
       ▼
  System of n first-order equations
```

---

## 3. Examples of Conversion

### Example 1: Second-Order Linear ODE

$$y'' + 5y' + 6y = 0$$

Let $x_1 = y$, $x_2 = y'$:

$$x_1' = x_2$$
$$x_2' = -6x_1 - 5x_2$$

In matrix form:

$$\begin{pmatrix} x_1 \\ x_2 \end{pmatrix}' = \begin{pmatrix} 0 & 1 \\ -6 & -5 \end{pmatrix} \begin{pmatrix} x_1 \\ x_2 \end{pmatrix}$$

### Example 2: Third-Order ODE

$$y''' - 3y'' + 2y' = e^t$$

Let $x_1 = y$, $x_2 = y'$, $x_3 = y''$:

$$x_1' = x_2, \quad x_2' = x_3, \quad x_3' = 3x_3 - 2x_2 + e^t$$

$$\mathbf{x}' = \begin{pmatrix} 0 & 1 & 0 \\ 0 & 0 & 1 \\ 0 & -2 & 3 \end{pmatrix} \mathbf{x} + \begin{pmatrix} 0 \\ 0 \\ e^t \end{pmatrix}$$

### Example 3: Coupled System (Two Masses)

```
  COUPLED SPRING-MASS SYSTEM

  Wall ─┤/\/\/├─[ m₁ ]─┤/\/\/├─[ m₂ ]─┤/\/\/├─ Wall
          k₁               k₂             k₃
```

$$m_1 x_1'' = -k_1 x_1 + k_2(x_2 - x_1)$$
$$m_2 x_2'' = -k_2(x_2 - x_1) - k_3 x_2$$

Let $u_1 = x_1$, $u_2 = x_1'$, $u_3 = x_2$, $u_4 = x_2'$:

$$u_1' = u_2$$
$$u_2' = \frac{-(k_1 + k_2)}{m_1} u_1 + \frac{k_2}{m_1} u_3$$
$$u_3' = u_4$$
$$u_4' = \frac{k_2}{m_2} u_1 + \frac{-(k_2 + k_3)}{m_2} u_3$$

---

## 4. Existence and Uniqueness

For $\mathbf{x}' = \mathbf{f}(t, \mathbf{x})$, $\mathbf{x}(t_0) = \mathbf{x}_0$:

If $\mathbf{f}$ and $\partial \mathbf{f}/\partial x_i$ are continuous in a region containing $(t_0, \mathbf{x}_0)$, then a **unique** solution exists on some interval $|t - t_0| < h$.

---

## 5. Superposition for Linear Systems

For $\mathbf{x}' = A\mathbf{x}$:

If $\mathbf{x}_1(t)$ and $\mathbf{x}_2(t)$ are solutions, then $c_1\mathbf{x}_1 + c_2\mathbf{x}_2$ is also a solution.

The general solution of an $n \times n$ system requires $n$ linearly independent solutions:

$$\mathbf{x}(t) = c_1\mathbf{x}_1(t) + c_2\mathbf{x}_2(t) + \cdots + c_n\mathbf{x}_n(t)$$

---

## 6. Applications Leading to Systems

| Application | Variables | System Size |
|------------|-----------|-------------|
| Predator-prey (Lotka-Volterra) | prey, predator populations | 2×2 |
| Coupled circuits | currents in loops | 2×2 or 3×3 |
| Multi-mass vibrations | displacements | 2n×2n |
| Chemical kinetics | concentrations | n×n |
| Epidemiology (SIR) | S, I, R populations | 3×3 |

```
  PREDATOR-PREY MODEL (LOTKA-VOLTERRA)

  dx/dt = αx - βxy     (prey growth - predation)
  dy/dt = δxy - γy     (predator growth - death)

  ┌──────────────────────────────┐
  │  Prey population x           │
  │    ╱╲    ╱╲    ╱╲            │
  │   ╱  ╲  ╱  ╲  ╱  ╲          │
  │──╱────╲╱────╲╱────╲──→ t    │
  │                               │
  │  Predator y (phase-shifted)   │
  │       ╱╲    ╱╲    ╱╲         │
  │      ╱  ╲  ╱  ╲  ╱  ╲       │
  │──── ╱────╲╱────╲╱────╲→ t   │
  └──────────────────────────────┘
```

---

## Summary Table

| Concept | Details |
|---------|---------|
| **System of ODEs** | Multiple coupled first-order equations |
| **Vector form** | $\mathbf{x}' = A\mathbf{x} + \mathbf{g}$ |
| **Conversion** | $n$-th order ODE → $n$ first-order equations |
| **State variables** | $x_1 = y, x_2 = y', \ldots$ |
| **Companion matrix** | Coefficients from the original ODE fill last row |
| **Superposition** | Valid for linear homogeneous systems |
| **General solution** | $n$ linearly independent vector solutions |

---

## Quick Revision Questions

1. Convert $y'' + 4y' + 3y = \sin t$ into a first-order system and write in matrix form.

2. Write the system of equations for the Lotka-Volterra predator-prey model. Under what conditions is it linear?

3. Convert the coupled system $x'' = -2x + y$, $y'' = x - 2y$ into a first-order system of four equations.

4. State the existence and uniqueness theorem for systems. How does it relate to the single-equation version?

5. Show that if $\mathbf{x}_1$ and $\mathbf{x}_2$ solve $\mathbf{x}' = A\mathbf{x}$, then so does $c_1\mathbf{x}_1 + c_2\mathbf{x}_2$.

6. Convert the RLC circuit equation $L\,q'' + R\,q' + \frac{1}{C}q = E(t)$ to a first-order system using $x_1 = q$ and $x_2 = i = q'$.

---

[← Previous: Beam Deflection](../07-Applications-Higher-Order/04-beam-deflection.md) | [Back to README](../README.md) | [Next: Matrix Method →](02-matrix-method.md)
