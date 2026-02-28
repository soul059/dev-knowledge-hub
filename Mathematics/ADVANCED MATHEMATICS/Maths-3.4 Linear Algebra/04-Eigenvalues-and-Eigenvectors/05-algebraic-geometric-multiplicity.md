# 4.5 Algebraic and Geometric Multiplicity

[← Previous: Properties of Eigenvalues](04-properties.md) · [Back to README](../README.md) · [Next: Unit 5 — Diagonalizable Matrices →](../05-Diagonalization/01-diagonalizable-matrices.md)

---

## Chapter Overview

The distinction between **algebraic** and **geometric** multiplicity determines whether a matrix can be diagonalised. This chapter defines both, proves the inequality between them, and builds towards the diagonalisation criterion.

---

## 1. Two Types of Multiplicity

> **Algebraic Multiplicity (AM):** The multiplicity of eigenvalue $\lambda$ as a root of the characteristic polynomial $p(\lambda) = \det(A - \lambda I)$.
>
> $$\text{am}(\lambda) = \text{multiplicity of } \lambda \text{ in } p(\lambda)$$

> **Geometric Multiplicity (GM):** The dimension of the eigenspace $E_\lambda = \ker(A - \lambda I)$.
>
> $$\text{gm}(\lambda) = \dim(E_\lambda) = n - \text{rank}(A - \lambda I)$$

---

## 2. The Fundamental Inequality

> **Theorem.** For every eigenvalue $\lambda$:
>
> $$1 \leq \text{gm}(\lambda) \leq \text{am}(\lambda)$$

```
  Multiplicity spectrum:
  
  1 ≤ gm(λ) ≤ am(λ) ≤ n
  │         │         │
  │         │         └─ Can't exceed matrix size
  │         └─ Geometric ≤ Algebraic (always)
  └─ At least 1 eigenvector exists
```

---

## 3. Examples Illustrating the Difference

### Example 1: GM = AM (nice case)

$$A = \begin{pmatrix} 3 & 0 \\ 0 & 3 \end{pmatrix}$$

$p(\lambda) = (3 - \lambda)^2$, so $\text{am}(3) = 2$.

$A - 3I = \begin{pmatrix} 0 & 0 \\ 0 & 0 \end{pmatrix}$, null space = $\mathbb{R}^2$, so $\text{gm}(3) = 2$.

GM = AM = 2. ✓ Matrix is diagonalisable (it's already diagonal).

### Example 2: GM < AM (deficient case)

$$A = \begin{pmatrix} 3 & 1 \\ 0 & 3 \end{pmatrix}$$

$p(\lambda) = (3 - \lambda)^2$, so $\text{am}(3) = 2$.

$A - 3I = \begin{pmatrix} 0 & 1 \\ 0 & 0 \end{pmatrix}$, null space = span$\{(1, 0)^T\}$, so $\text{gm}(3) = 1$.

GM = 1 < AM = 2. ✗ Matrix is **not** diagonalisable.

### Example 3: $3 \times 3$ with mixed multiplicities

$$A = \begin{pmatrix} 2 & 1 & 0 \\ 0 & 2 & 0 \\ 0 & 0 & 5 \end{pmatrix}$$

$p(\lambda) = (2 - \lambda)^2(5 - \lambda)$

| Eigenvalue | am | gm | 
|:---:|:---:|:---:|
| $\lambda = 2$ | 2 | $\dim\ker\begin{pmatrix} 0 & 1 & 0 \\ 0 & 0 & 0 \\ 0 & 0 & 3 \end{pmatrix} = 1$ |
| $\lambda = 5$ | 1 | 1 |

Total eigenvectors: 2 (but need 3 for diagonalisation). **Not diagonalisable.**

### Example 4: $3 \times 3$ where GM = AM everywhere

$$A = \begin{pmatrix} 2 & 0 & 0 \\ 0 & 2 & 0 \\ 0 & 0 & 5 \end{pmatrix}$$

| Eigenvalue | am | gm |
|:---:|:---:|:---:|
| $\lambda = 2$ | 2 | 2 |
| $\lambda = 5$ | 1 | 1 |

Total eigenvectors: 3 = $n$. **Diagonalisable** ✓

---

## 4. The Diagonalisation Criterion

> **Theorem.** An $n \times n$ matrix $A$ is diagonalisable if and only if:
>
> $$\text{gm}(\lambda_i) = \text{am}(\lambda_i) \quad \text{for every eigenvalue } \lambda_i$$
>
> Equivalently: $A$ has $n$ linearly independent eigenvectors.

```
  Decision tree for diagonalisability:
  
  ┌──────────────────────┐
  │  Find eigenvalues    │
  └──────────┬───────────┘
             ▼
  ┌──────────────────────┐    Yes ───► Diagonalisable
  │ n distinct eigenvals? │──────────  (always works)
  └──────────┬───────────┘
             │ No (repeated eigenvalues)
             ▼
  ┌──────────────────────────────────┐
  │ For each repeated eigenvalue:    │
  │ Check if gm(λ) = am(λ)?         │
  └──────────────┬───────────────────┘
                 │
           ┌─────┴─────┐
       All yes?     Some gm < am?
           │              │
     Diagonalisable   NOT diagonalisable
```

---

## 5. Computing Geometric Multiplicity

$$\text{gm}(\lambda) = n - \text{rank}(A - \lambda I)$$

Steps:
1. Form $A - \lambda I$
2. Row reduce to echelon form
3. Count pivot columns → rank
4. $\text{gm}(\lambda) = n - \text{rank}$

**Worked Example:**

$$A = \begin{pmatrix} 5 & 4 & 2 \\ 4 & 5 & 2 \\ 2 & 2 & 2 \end{pmatrix}$$

$p(\lambda) = -(\lambda - 10)(\lambda - 1)^2$

For $\lambda = 1$ ($\text{am} = 2$):

$$A - I = \begin{pmatrix} 4 & 4 & 2 \\ 4 & 4 & 2 \\ 2 & 2 & 1 \end{pmatrix} \xrightarrow{\text{RREF}} \begin{pmatrix} 1 & 1 & 1/2 \\ 0 & 0 & 0 \\ 0 & 0 & 0 \end{pmatrix}$$

rank = 1, so $\text{gm}(1) = 3 - 1 = 2 = \text{am}(1)$. ✓ Diagonalisable.

---

## 6. Summary of Key Relationships

| Condition | Consequence |
|-----------|------------|
| $n$ distinct eigenvalues | Always diagonalisable |
| All $\text{gm}(\lambda_i) = \text{am}(\lambda_i)$ | Diagonalisable |
| Some $\text{gm}(\lambda_i) < \text{am}(\lambda_i)$ | **Not** diagonalisable |
| $A$ is symmetric ($A = A^T$) | Always diagonalisable (spectral theorem) |
| $\sum \text{am}(\lambda_i) < n$ | Complex eigenvalues exist (for real matrices) |
| $\sum \text{gm}(\lambda_i) = n$ | Diagonalisable |

---

## 7. Jordan Preview

When GM < AM, the matrix cannot be diagonalised, but it can be put in **Jordan normal form**:

$$J = \begin{pmatrix} \lambda & 1 & & \\ & \lambda & 1 & \\ & & \ddots & 1 \\ & & & \lambda \end{pmatrix}$$

The 1's above the diagonal (called "Jordan chains") fill the gap between geometric and algebraic multiplicity. This is covered in advanced courses.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Algebraic multiplicity | Multiplicity of $\lambda$ in $p(\lambda)$ |
| Geometric multiplicity | $\dim(E_\lambda) = \dim\ker(A - \lambda I)$ |
| Fundamental inequality | $1 \leq \text{gm} \leq \text{am}$ |
| Computing GM | $\text{gm} = n - \text{rank}(A - \lambda I)$ |
| Diagonalisable iff | $\text{gm} = \text{am}$ for all eigenvalues |
| Distinct eigenvalues | Automatically diagonalisable |
| Symmetric matrices | Always diagonalisable |
| GM < AM | Need Jordan form (not diagonalisable) |

---

## Quick Revision Questions

1. Find the algebraic and geometric multiplicities for all eigenvalues of $A = \begin{pmatrix} 4 & 1 \\ 0 & 4 \end{pmatrix}$.

2. Give an example of a $3 \times 3$ matrix with eigenvalue $\lambda = 2$ of AM = 3 and GM = 1.

3. Why must $\text{gm}(\lambda) \geq 1$ for any eigenvalue?

4. A $4 \times 4$ matrix has characteristic polynomial $(\lambda - 1)^2(\lambda - 3)^2$. Under what conditions is it diagonalisable?

5. Prove that symmetric matrices always satisfy $\text{gm} = \text{am}$ for every eigenvalue.

6. Show that if $A$ is $n \times n$ with $n$ distinct eigenvalues, then $A$ is diagonalisable.

---

[← Previous: Properties of Eigenvalues](04-properties.md) · [Back to README](../README.md) · [Next: Unit 5 — Diagonalizable Matrices →](../05-Diagonalization/01-diagonalizable-matrices.md)
