# Chapter 3.6: Relaxation Methods

[← Previous: Gauss-Seidel](05-gauss-seidel.md) | [Next: Unit 4 — Finite Differences →](../04-Interpolation/01-finite-differences.md)

---

## Overview

**Relaxation methods** accelerate iterative convergence by introducing a **relaxation parameter** $\omega$. The most important is the **Successive Over-Relaxation (SOR)** method, which generalizes Gauss-Seidel by taking a weighted average of old and new values. Choosing the right $\omega$ can dramatically speed up convergence.

---

## 1. SOR Iteration Formula

$$x_i^{(k+1)} = (1 - \omega) x_i^{(k)} + \frac{\omega}{a_{ii}} \left( b_i - \sum_{j=1}^{i-1} a_{ij} x_j^{(k+1)} - \sum_{j=i+1}^{n} a_{ij} x_j^{(k)} \right)$$

Or equivalently:

$$x_i^{(k+1)} = x_i^{(k)} + \omega \left( x_i^{GS} - x_i^{(k)} \right)$$

where $x_i^{GS}$ is the Gauss-Seidel update.

```
  ω < 1:  Under-relaxation   (takes smaller step than GS)
  ω = 1:  Gauss-Seidel       (standard)
  ω > 1:  Over-relaxation    (takes larger step — accelerates!)

  Convergence requires: 0 < ω < 2

  ←─── Under ───┼─── GS ───┼─── Over ──→
                 0          1           2
                      ω
```

---

## 2. Geometric Intuition

```
  x(k+1)
    │
    │              ● ω > 1 (overshoot past GS)
    │             ╱
    │            ╱
    │           ● ω = 1 (Gauss-Seidel point)
    │          ╱
    │         ╱
    │        ● ω < 1 (conservative step)
    │       ╱
    │      ╱
    │─────● x(k) (current value)
    │
    └───────────────────────── iterations
```

---

## 3. Choosing Optimal $\omega$

For certain matrices (e.g., tridiagonal, block tridiagonal from discretized PDEs):

$$\omega_{\text{opt}} = \frac{2}{1 + \sqrt{1 - \rho_J^2}}$$

where $\rho_J$ is the spectral radius of the Jacobi iteration matrix.

### Effect of $\omega$ on Convergence

| $\omega$ | Spectral Radius $\rho_{SOR}$ | Convergence |
|:--------:|:----------------------------:|:-----------:|
| $0.5$ | Slow | Under-relaxation |
| $1.0$ | $\rho_{GS}$ | Gauss-Seidel |
| $\omega_{opt}$ | Minimum! | Fastest |
| $1.9$ | May be large | May oscillate |
| $\geq 2$ | $\geq 1$ | Diverges |

```
  Spectral radius ρ(ω)
    │
  1 ┤─────╲                        ╱─────
    │      ╲                      ╱
    │       ╲                    ╱
    │        ╲                  ╱
    │         ╲               ╱
    │          ╲             ╱
    │           ╲    ●      ╱
    │            ╲  minimum╱
    │             ╲  ╱    ╱
    │              ╳     ╱
  0 ┤               ω_opt
    ├──┤──┤──┤──┤──┤──┤──┤──┤
    0  0.5  1.0     ω_opt  2.0  ω
```

---

## 4. Worked Example

Consider a system where the Jacobi spectral radius is $\rho_J = 0.9$.

**Gauss-Seidel**: $\rho_{GS} \approx \rho_J^2 = 0.81$
**Optimal SOR**: 

$$\omega_{opt} = \frac{2}{1 + \sqrt{1 - 0.81}} = \frac{2}{1 + \sqrt{0.19}} = \frac{2}{1.436} \approx 1.393$$

$$\rho_{SOR}(\omega_{opt}) = \omega_{opt} - 1 = 0.393$$

### Iterations to reduce error by $10^{-6}$:

$$n = \frac{\ln(10^{-6})}{\ln(\rho)}$$

| Method | $\rho$ | Iterations |
|--------|:------:|:----------:|
| Jacobi | 0.9 | 131 |
| Gauss-Seidel | 0.81 | 66 |
| SOR ($\omega_{opt}$) | 0.393 | 15 |

**SOR is nearly 9× faster than Jacobi!**

---

## 5. Types of Relaxation

| Type | $\omega$ | Use Case |
|------|:--------:|----------|
| **Under-relaxation** | $0 < \omega < 1$ | When GS diverges or oscillates; stabilizes |
| **Gauss-Seidel** | $\omega = 1$ | Standard iterative method |
| **Over-relaxation (SOR)** | $1 < \omega < 2$ | Accelerate convergence |
| **SSOR** | Alternating | Symmetric SOR; used as preconditioner |

---

## 6. Convergence Theorem

**Kahan's Theorem**: For SOR to converge, it is **necessary** that $0 < \omega < 2$.

**Ostrowski-Reich Theorem**: If $A$ is symmetric positive definite and $0 < \omega < 2$, then SOR converges for any initial guess.

---

## 7. Real-World Applications

| Application | Role of SOR |
|-------------|------------|
| PDE solvers (Laplace, Poisson) | SOR is the classical fast solver |
| Computational fluid dynamics | Pressure correction equations |
| Image reconstruction | Iterative algorithms for CT/MRI |
| Game physics | Contact force resolution |
| Finite difference methods | Grid-based PDE solutions |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Formula** | $x_i^{(k+1)} = (1-\omega)x_i^{(k)} + \omega \cdot x_i^{GS}$ |
| **$\omega = 1$** | Reduces to Gauss-Seidel |
| **Optimal $\omega$** | $\frac{2}{1+\sqrt{1-\rho_J^2}}$ |
| **Convergence requires** | $0 < \omega < 2$ |
| **For SPD matrices** | Always converges if $0 < \omega < 2$ |
| **Speed-up over GS** | Can be dramatic (5-10×) |

---

## Quick Revision Questions

1. **Write the SOR iteration formula. What does it reduce to when $\omega = 1$?**

2. **Why must $\omega$ be in $(0, 2)$ for convergence?**

3. **If the Jacobi spectral radius is 0.95, compute the optimal $\omega$ and compare iterations needed for Jacobi, GS, and SOR to achieve $10^{-8}$ accuracy.**

4. **When would you use under-relaxation ($\omega < 1$) instead of over-relaxation?**

5. **State the Ostrowski-Reich theorem.**

6. **In a PDE discretization producing a tridiagonal system, why is SOR particularly effective?**

---

[← Previous: Gauss-Seidel](05-gauss-seidel.md) | [Next: Unit 4 — Finite Differences →](../04-Interpolation/01-finite-differences.md)
