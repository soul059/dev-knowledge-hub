# Chapter 3.5: Gauss-Seidel Method

[← Previous: Jacobi Method](04-jacobi-method.md) | [Next: Relaxation Methods →](06-relaxation-methods.md)

---

## Overview

The **Gauss-Seidel method** improves upon Jacobi by using **updated values as soon as they are available**. When computing $x_i^{(k+1)}$, it uses the already-computed values $x_1^{(k+1)}, \ldots, x_{i-1}^{(k+1)}$ rather than all old values from iteration $k$. This typically converges **faster** than Jacobi.

---

## 1. Iteration Formula

$$x_i^{(k+1)} = \frac{1}{a_{ii}} \left( b_i - \sum_{j=1}^{i-1} a_{ij} x_j^{(k+1)} - \sum_{j=i+1}^{n} a_{ij} x_j^{(k)} \right)$$

Notice: the first sum uses **new** values ($k+1$), the second uses **old** values ($k$).

### Matrix Form

$$x^{(k+1)} = (D + L_0)^{-1}\left(b - U_0 x^{(k)}\right)$$

---

## 2. Jacobi vs. Gauss-Seidel

```
  Jacobi:                          Gauss-Seidel:

  x₁(k+1) uses: x₂(k), x₃(k)    x₁(k+1) uses: x₂(k), x₃(k)
  x₂(k+1) uses: x₁(k), x₃(k)    x₂(k+1) uses: x₁(k+1), x₃(k)  ← UPDATED!
  x₃(k+1) uses: x₁(k), x₂(k)    x₃(k+1) uses: x₁(k+1), x₂(k+1) ← BOTH UPDATED!

  Jacobi needs TWO arrays          Gauss-Seidel needs only ONE array
  (old and new)                    (overwrite in place)
```

---

## 3. Worked Example

Same system as Jacobi chapter:
$$10x_1 + 2x_2 + x_3 = 9$$
$$x_1 + 5x_2 - x_3 = 4$$
$$2x_1 + 3x_2 + 10x_3 = 22$$

Starting with $x^{(0)} = [0, 0, 0]^T$:

**Iteration 1:**
$$x_1^{(1)} = \frac{9 - 2(0) - 1(0)}{10} = 0.9000$$
$$x_2^{(1)} = \frac{4 - (0.9) + (0)}{5} = 0.6200$$
$$x_3^{(1)} = \frac{22 - 2(0.9) - 3(0.62)}{10} = 1.8340$$

**Iteration 2:**
$$x_1^{(2)} = \frac{9 - 2(0.62) - 1(1.834)}{10} = 0.5926$$
$$x_2^{(2)} = \frac{4 - (0.5926) + (1.834)}{5} = 1.0483$$
$$x_3^{(2)} = \frac{22 - 2(0.5926) - 3(1.0483)}{10} = 1.7671$$

| $k$ | $x_1$ | $x_2$ | $x_3$ | $\|x^{(k+1)}-x^{(k)}\|_\infty$ |
|:---:|:-----:|:-----:|:-----:|:-----:|
| 0 | 0.0000 | 0.0000 | 0.0000 | — |
| 1 | 0.9000 | 0.6200 | 1.8340 | 1.8340 |
| 2 | 0.5926 | 1.0483 | 1.7671 | 0.4283 |
| 3 | 0.5803 | 0.8374 | 2.0485 | 0.2813 |
| 4 | 0.6325 | 0.8832 | 1.9893 | 0.0593 |
| 5 | 0.6234 | 0.8732 | 2.0010 | 0.0100 |

Compare with Jacobi: Gauss-Seidel converges faster (fewer iterations to same accuracy).

---

## 4. Convergence

### Sufficient Conditions

1. **Strict diagonal dominance** (same as Jacobi) → guaranteed convergence
2. **$A$ is symmetric positive definite** → guaranteed convergence
3. **Spectral radius**: $\rho((D+L_0)^{-1}U_0) < 1$

### Comparison with Jacobi

| Property | Jacobi | Gauss-Seidel |
|----------|:------:|:------------:|
| Spectral radius | $\rho_J$ | Typically $\rho_{GS} \approx \rho_J^2$ |
| Convergence speed | Baseline | ~2× faster (roughly) |
| Storage | 2 arrays | 1 array |
| Parallelizable | Yes | No (sequential dependency) |

### Stein-Rosenberg Theorem

For matrices with positive diagonal and non-positive off-diagonal entries:
- Either both Jacobi and Gauss-Seidel converge (GS faster), or both diverge, or
- $\rho_{GS} = \rho_J^2$ (GS converges in half the iterations)

---

## 5. When Gauss-Seidel Fails

Gauss-Seidel is NOT guaranteed to converge for all systems. Example:

$$A = \begin{bmatrix} 1 & 2 \\ 2 & 1 \end{bmatrix}$$

This is NOT diagonally dominant ($|1| < |2|$). Both Jacobi and Gauss-Seidel **diverge**.

---

## 6. Real-World Applications

| Application | Why Gauss-Seidel |
|-------------|-----------------|
| Power flow analysis | Large sparse systems from power grids |
| CFD (computational fluid dynamics) | Iterative PDE solvers |
| Finite element analysis | Sparse stiffness matrices |
| Game physics engines | Constraint solving |
| Sudoku solvers | Constraint propagation is analogous |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Type** | Iterative method |
| **Innovation** | Uses updated values immediately |
| **Formula** | Uses $x_j^{(k+1)}$ for $j < i$, $x_j^{(k)}$ for $j > i$ |
| **Converges if** | Diag. dominant or symmetric positive definite |
| **Speed vs Jacobi** | Typically ~2× faster |
| **Storage** | In-place (single array) |
| **Parallelizable** | Not directly (sequential dependencies) |

---

## Quick Revision Questions

1. **How does Gauss-Seidel differ from Jacobi? Show with the formula.**

2. **Solve $5x + y = 6$, $x + 4y = 9$ using Gauss-Seidel (3 iterations, start from $[0,0]$).**

3. **Why is Gauss-Seidel generally faster than Jacobi?**

4. **State the Stein-Rosenberg theorem. What does it imply about the relative convergence of Jacobi and Gauss-Seidel?**

5. **Why can't Gauss-Seidel be easily parallelized?**

6. **For what types of matrices is Gauss-Seidel guaranteed to converge?**

---

[← Previous: Jacobi Method](04-jacobi-method.md) | [Next: Relaxation Methods →](06-relaxation-methods.md)
