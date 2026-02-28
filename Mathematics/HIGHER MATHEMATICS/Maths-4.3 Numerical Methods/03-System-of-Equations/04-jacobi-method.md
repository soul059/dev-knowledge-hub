# Chapter 3.4: Jacobi Method

[← Previous: LU Decomposition](03-lu-decomposition.md) | [Next: Gauss-Seidel Method →](05-gauss-seidel.md)

---

## Overview

The **Jacobi method** is an **iterative** technique for solving $Ax = b$. Unlike direct methods (Gaussian elimination, LU), iterative methods start with an initial guess and refine it repeatedly. They are especially useful for **large, sparse systems** where direct methods are too expensive.

---

## 1. Idea: Splitting the Matrix

Decompose $A = D + L_0 + U_0$ where:
- $D$ = diagonal of $A$
- $L_0$ = strictly lower triangular part
- $U_0$ = strictly upper triangular part

From $Ax = b$: $(D + L_0 + U_0)x = b$

$$Dx = b - (L_0 + U_0)x$$

$$\boxed{x^{(k+1)} = D^{-1}\left(b - (L_0 + U_0)x^{(k)}\right)}$$

### Component Form

$$x_i^{(k+1)} = \frac{1}{a_{ii}}\left(b_i - \sum_{\substack{j=1 \\ j \neq i}}^{n} a_{ij} x_j^{(k)}\right), \quad i = 1, 2, \ldots, n$$

**Key**: ALL values on the right side come from the **previous** iteration $k$.

---

## 2. Algorithm

```
  ┌────────────────────────────┐
  │ Input: A, b, x⁰, ε, Nmax  │
  └─────────────┬──────────────┘
                ▼
  ┌────────────────────────────┐
  │  For each i = 1,2,...,n:   │◄──────────┐
  │    xᵢ(k+1) = (bᵢ -        │           │
  │     Σⱼ≠ᵢ aᵢⱼxⱼ(k)) / aᵢᵢ │           │
  └─────────────┬──────────────┘           │
                ▼                           │
  ┌────────────────────────────┐           │
  │ ‖x(k+1) - x(k)‖ < ε ?    │           │
  └───┬────────────────┬───────┘           │
    Yes                No ─────────────────┘
      ▼
  ┌───────────────┐
  │ Output x(k+1) │
  └───────────────┘
```

---

## 3. Worked Example

Solve:
$$10x_1 + 2x_2 + x_3 = 9$$
$$x_1 + 5x_2 - x_3 = 4$$
$$2x_1 + 3x_2 + 10x_3 = 22$$

Jacobi iterations:

$$x_1^{(k+1)} = \frac{9 - 2x_2^{(k)} - x_3^{(k)}}{10}$$

$$x_2^{(k+1)} = \frac{4 - x_1^{(k)} + x_3^{(k)}}{5}$$

$$x_3^{(k+1)} = \frac{22 - 2x_1^{(k)} - 3x_2^{(k)}}{10}$$

Starting with $x^{(0)} = [0, 0, 0]^T$:

| $k$ | $x_1$ | $x_2$ | $x_3$ | $\|x^{(k+1)}-x^{(k)}\|_\infty$ |
|:---:|:-----:|:-----:|:-----:|:-----:|
| 0 | 0.0000 | 0.0000 | 0.0000 | — |
| 1 | 0.9000 | 0.8000 | 2.2000 | 2.2000 |
| 2 | 0.5600 | 0.8600 | 1.9600 | 0.3400 |
| 3 | 0.6280 | 0.8800 | 2.0020 | 0.0680 |
| 4 | 0.6240 | 0.8748 | 1.9920 | 0.0100 |
| 5 | 0.6250 | 0.8736 | 1.9983 | 0.0063 |
| ... | ... | ... | ... | ... |
| ∞ | 0.625 | 0.875 | 2.000 | 0 |

**Converges to $x = [0.625, 0.875, 2.0]^T$.**

---

## 4. Convergence Conditions

The Jacobi method converges if **any** of these conditions hold:

### Sufficient Condition 1: Diagonal Dominance

$A$ is **strictly diagonally dominant** if:

$$|a_{ii}| > \sum_{\substack{j=1 \\ j \neq i}}^{n} |a_{ij}| \quad \text{for all } i$$

```
  Example (diagonally dominant):
  ┌              ┐
  │ 10   2   1   │   |10| > |2|+|1| = 3  ✓
  │  1   5  -1   │   |5|  > |1|+|1| = 2  ✓
  │  2   3  10   │   |10| > |2|+|3| = 5  ✓
  └              ┘
```

### Sufficient Condition 2: Spectral Radius

$$\rho(D^{-1}(L_0 + U_0)) < 1$$

where $\rho$ denotes the spectral radius (largest eigenvalue magnitude).

---

## 5. Parallelism

A major advantage of Jacobi: **all components update independently**.

```
  Jacobi (parallelizable):

  x₁(k+1) ─────┐
  x₂(k+1) ────┐│  All computed SIMULTANEOUSLY
  x₃(k+1) ───┐││  using x(k) only
              │││
              ▼▼▼
  ┌─────────────────┐
  │  All use x(k)   │   ← no dependency between components
  └─────────────────┘

  Gauss-Seidel (sequential):

  x₁(k+1) ───► x₂(k+1) ───► x₃(k+1)
  (uses x(k))  (uses x₁(k+1)) (uses x₁(k+1), x₂(k+1))
```

---

## 6. Real-World Applications

| Application | Why Jacobi |
|-------------|-----------|
| Image processing | Pixel updates independent (Poisson editing) |
| Parallel computing / GPU | Perfect for massively parallel hardware |
| Heat conduction (PDE) | Grid updates can be parallelized |
| PageRank (simplified) | Web page scores updated simultaneously |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Type** | Iterative method |
| **Formula** | $x_i^{(k+1)} = \frac{1}{a_{ii}}(b_i - \sum_{j \neq i} a_{ij}x_j^{(k)})$ |
| **Converges if** | $A$ is strictly diagonally dominant |
| **Rate** | Linear; depends on spectral radius |
| **Cost per iteration** | $O(n^2)$ (or $O(\text{nnz})$ for sparse) |
| **Parallelizable** | Yes — all components update independently |
| **Initial guess** | Any; $x^{(0)} = 0$ is common |

---

## Quick Revision Questions

1. **Write the Jacobi iteration formula in both matrix and component form.**

2. **What is diagonal dominance? Why does it guarantee convergence of Jacobi?**

3. **Solve $4x + y = 7$, $x + 3y = 11$ using Jacobi method (3 iterations, $x^{(0)} = [0,0]$).**

4. **Why is Jacobi well-suited for parallel computing?**

5. **Give an example of a system where Jacobi does NOT converge.**

6. **How does the spectral radius relate to convergence speed?**

---

[← Previous: LU Decomposition](03-lu-decomposition.md) | [Next: Gauss-Seidel Method →](05-gauss-seidel.md)
