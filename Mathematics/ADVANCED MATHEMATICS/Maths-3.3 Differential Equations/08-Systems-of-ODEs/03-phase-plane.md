# Chapter 8.3: Phase Plane Analysis

[← Previous: Matrix Method](02-matrix-method.md) | [Back to README](../README.md) | [Next: Power Series Method →](../09-Series-Solutions/01-power-series-method.md)

---

## Overview

The **phase plane** is a graphical tool for analyzing 2×2 autonomous systems $\mathbf{x}' = A\mathbf{x}$. Instead of plotting $x(t)$ vs $t$, we plot trajectories in the $(x_1, x_2)$-plane, revealing the qualitative behavior at a glance.

---

## 1. Phase Portrait Basics

For the system $x_1' = f(x_1, x_2)$, $x_2' = g(x_1, x_2)$:

- A **trajectory** (orbit) is a curve $(x_1(t), x_2(t))$ in the phase plane
- At each point, the vector $(f, g)$ gives the direction of motion
- **Critical points** (equilibria): where $f = 0$ and $g = 0$ simultaneously

For the linear system $\mathbf{x}' = A\mathbf{x}$, the **origin is the only critical point** (if $\det A \neq 0$).

---

## 2. Classification by Eigenvalues

### Complete Classification

| Eigenvalues | Type | Stability | Phase Portrait |
|------------|------|-----------|---------------|
| $\lambda_1, \lambda_2 < 0$ (distinct real) | **Stable node** | Asymptotically stable | All trajectories → origin |
| $\lambda_1, \lambda_2 > 0$ (distinct real) | **Unstable node** | Unstable | All trajectories ← origin |
| $\lambda_1 < 0 < \lambda_2$ (real, opposite signs) | **Saddle point** | Unstable | Hyperbolic trajectories |
| $\alpha \pm i\beta$, $\alpha < 0$ | **Stable spiral** | Asymptotically stable | Spirals into origin |
| $\alpha \pm i\beta$, $\alpha > 0$ | **Unstable spiral** | Unstable | Spirals away from origin |
| $\pm i\beta$ (pure imaginary) | **Center** | Stable (not asymp.) | Closed ellipses |
| $\lambda_1 = \lambda_2 < 0$, 2 eigenvectors | **Stable star node** | Asymptotically stable | Straight lines to origin |
| $\lambda_1 = \lambda_2 < 0$, 1 eigenvector | **Stable degenerate node** | Asymptotically stable | Tangent curves to origin |

---

## 3. Phase Portrait Gallery

```
  STABLE NODE                    UNSTABLE NODE
  (λ₁, λ₂ < 0)                  (λ₁, λ₂ > 0)
                                 
       ╲  │  ╱                        ╲  │  ╱
        ╲ │ ╱                          ↖ │ ↗
         ╲│╱                            ╲│╱
    ──────●──────                  ──────●──────
         ╱│╲                            ╱│╲
        ╱ │ ╲                          ↙ │ ↘
       ╱  │  ╲                        ╱  │  ╲
   (arrows point IN)            (arrows point OUT)


  SADDLE POINT                   CENTER
  (λ₁ < 0 < λ₂)                 (λ = ±iβ)
                                 
    ↗     │     ↖                  ╭──────╮
     ╲    │    ╱                   │  ╭──╮│
      ╲   │   ╱                    │  │  ││
    ───╲──●──╱───                  │  │ ●││
      ╱   │   ╲                    │  ╰──╯│
     ╱    │    ╲                   ╰──────╯
    ↙     │     ↘              (closed orbits)


  STABLE SPIRAL                  UNSTABLE SPIRAL
  (α < 0, β ≠ 0)                (α > 0, β ≠ 0)
                                 
      ╭─╮                            ╭───╮
     ╭│ ╰─╮                        ╭─╯ ╭╯
     │╰╮  │                       ╭╯╭─╯
     │ ●╯ │                       │ ● ╮
     ╰─╮ ╭╯                       ╰╮╰─╮
       ╰─╯                          ╰──╯
   (spirals IN)                 (spirals OUT)


  STAR NODE                      DEGENERATE NODE
  (λ repeated, 2 eigvec)        (λ repeated, 1 eigvec)
                                 
       ╲  │  ╱                         │
        ╲ │ ╱                        ╲ │ ╱
    ─────╲│╱──────                  ──╲│╱──
    ──────●───────                  ───●───
    ─────╱│╲──────                  ──╱│╲──
        ╱ │ ╲                        ╱ │ ╲
       ╱  │  ╲                         │
   (all directions)             (one tangent line)
```

---

## 4. Trace-Determinant Plane

For $2 \times 2$ matrix $A$, let $T = \text{tr}(A) = a_{11} + a_{22}$ and $D = \det(A)$:

$$\lambda = \frac{T \pm \sqrt{T^2 - 4D}}{2}$$

```
  TRACE-DETERMINANT CLASSIFICATION
                     
           D (det)
           │
           │  Stable    │  Unstable
           │  Spiral    │  Spiral
           │            │
           │     Center │
    ───────│─────●──────│────────── T² = 4D (parabola)
           │   ╱    ╲   │
    Stable │  ╱      ╲  │ Unstable
    Node   │ ╱        ╲ │ Node
           │╱          ╲│
    ═══════●════════════●═════ T (trace)
          ╱│╲          ╱│╲
         ╱ │ ╲        ╱ │ ╲
           │    Saddle   │
           │   Point     │
    ───────│─────────────│────
           │ (D < 0)     │
```

| Region | Classification |
|--------|---------------|
| $D < 0$ | Saddle point |
| $D > 0$, $T < 0$, $T^2 > 4D$ | Stable node |
| $D > 0$, $T > 0$, $T^2 > 4D$ | Unstable node |
| $D > 0$, $T < 0$, $T^2 < 4D$ | Stable spiral |
| $D > 0$, $T > 0$, $T^2 < 4D$ | Unstable spiral |
| $D > 0$, $T = 0$ | Center |
| $T^2 = 4D$, $T \neq 0$ | Degenerate/star node |

---

## 5. Worked Examples

### Example 1: Classify and Sketch

$$\mathbf{x}' = \begin{pmatrix} -3 & 0 \\ 0 & -1 \end{pmatrix}\mathbf{x}$$

$\lambda_1 = -3$, $\lambda_2 = -1$ (both negative, distinct) → **Stable node**

Eigenvectors: $\mathbf{v}_1 = \begin{pmatrix} 1 \\ 0 \end{pmatrix}$ (fast direction), $\mathbf{v}_2 = \begin{pmatrix} 0 \\ 1 \end{pmatrix}$ (slow direction)

As $t \to \infty$, trajectories approach origin tangent to the **slow eigenvector** ($\lambda = -1$).

### Example 2: Damped Oscillator

$$x'' + 2x' + 5x = 0 \implies \mathbf{x}' = \begin{pmatrix} 0 & 1 \\ -5 & -2 \end{pmatrix}\mathbf{x}$$

$T = -2$, $D = 5$, $T^2 - 4D = 4 - 20 = -16 < 0$

Since $T < 0$ and $T^2 < 4D$ → **Stable spiral**

$\lambda = -1 \pm 2i$ (decay rate $\alpha = -1$, oscillation frequency $\beta = 2$)

### Example 3: Population Competition

$$x' = x(3 - x - y), \quad y' = y(2 - x - y)$$

Critical points: $(0,0)$, $(3,0)$, $(0,2)$

At $(0,0)$: Jacobian $= \begin{pmatrix} 3 & 0 \\ 0 & 2 \end{pmatrix}$ → Unstable node

At $(3,0)$: Jacobian $= \begin{pmatrix} -3 & -3 \\ 0 & -1 \end{pmatrix}$ → Stable node ($\lambda = -3, -1$)

At $(0,2)$: Jacobian $= \begin{pmatrix} 1 & 0 \\ -2 & -2 \end{pmatrix}$ → Saddle ($\lambda = 1, -2$)

---

## 6. Stability Summary

$$\text{Asymptotically stable} \iff \text{all eigenvalues have Re}(\lambda) < 0$$

$$\text{Stable} \iff \text{Re}(\lambda) \leq 0 \text{ (and if } \text{Re}(\lambda) = 0 \text{, eigenspace is complete)}$$

$$\text{Unstable} \iff \text{any eigenvalue has Re}(\lambda) > 0$$

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Phase plane** | Plot trajectories in $(x_1, x_2)$-plane |
| **Critical point** | Where $\mathbf{f} = \mathbf{0}$ |
| **Node** | All trajectories approach/leave origin |
| **Saddle** | Some approach, some leave |
| **Spiral** | Oscillatory approach/departure |
| **Center** | Closed orbits, no approach |
| **Trace-det test** | Quick classification from $T$, $D$ |
| **Stability criterion** | Re($\lambda$) < 0 for all eigenvalues |

---

## Quick Revision Questions

1. Classify the critical point at the origin for $\mathbf{x}' = \begin{pmatrix} 1 & -5 \\ 1 & -1 \end{pmatrix}\mathbf{x}$.

2. A 2×2 system has $\text{tr}(A) = -4$ and $\det(A) = 5$. Classify the critical point and determine stability.

3. Sketch the phase portrait for a saddle point with eigenvectors $\begin{pmatrix} 1 \\ 1 \end{pmatrix}$ ($\lambda = 2$) and $\begin{pmatrix} 1 \\ -1 \end{pmatrix}$ ($\lambda = -3$).

4. Why can a center never be asymptotically stable?

5. For the system $x' = -y$, $y' = x$, find the eigenvalues, classify the critical point, and describe the trajectories.

6. A nonlinear system has Jacobian $J = \begin{pmatrix} -1 & 2 \\ -2 & -1 \end{pmatrix}$ at an equilibrium. Classify the equilibrium and determine stability.

---

[← Previous: Matrix Method](02-matrix-method.md) | [Back to README](../README.md) | [Next: Power Series Method →](../09-Series-Solutions/01-power-series-method.md)
