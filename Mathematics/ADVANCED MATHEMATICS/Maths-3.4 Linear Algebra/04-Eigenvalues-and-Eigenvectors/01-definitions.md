# 4.1 Definitions: Eigenvalues and Eigenvectors

[← Previous: Unit 3 — Orthonormal Basis](../03-Inner-Product-Spaces/06-orthonormal-basis.md) · [Back to README](../README.md) · [Next: Characteristic Polynomial →](02-characteristic-polynomial.md)

---

## Chapter Overview

Eigenvalues and eigenvectors reveal the **intrinsic geometry** of a linear transformation — the directions that are merely scaled (not rotated) by the map. This chapter introduces the fundamental definitions, geometric intuition, and basic examples.

---

## 1. Motivation

Consider a $2 \times 2$ matrix acting on vectors in $\mathbb{R}^2$. Most vectors change direction when multiplied by $A$. But some special vectors $\mathbf{v}$ satisfy:

$$A\mathbf{v} = \lambda \mathbf{v}$$

These vectors stay on their original line through the origin — they are only **scaled** by factor $\lambda$.

```
  General vector:              Eigenvector:
  
       A·v                         A·v = λv
        ↗                            ↑
       /                             |
      /                              |   (same direction,
     ·──→ v                          ·──→ v   scaled by λ)
  
  Direction changes             Direction preserved
```

---

## 2. Formal Definitions

> **Definition.** Let $A$ be an $n \times n$ matrix. A scalar $\lambda$ is an **eigenvalue** of $A$ if there exists a nonzero vector $\mathbf{v}$ such that:
>
> $$A\mathbf{v} = \lambda \mathbf{v}$$
>
> The nonzero vector $\mathbf{v}$ is called an **eigenvector** corresponding to $\lambda$.

Key points:
- $\mathbf{v} \neq \mathbf{0}$ (the zero vector is never an eigenvector)
- $\lambda$ can be zero, positive, negative, or complex
- "Eigen" is German for "own" or "characteristic"

Alternative terminologies: characteristic value/vector, proper value/vector, latent root/vector.

---

## 3. Eigenspace

> **Definition.** The **eigenspace** of $A$ corresponding to eigenvalue $\lambda$ is:
>
> $$E_\lambda = \ker(A - \lambda I) = \{\mathbf{v} \in \mathbb{R}^n : A\mathbf{v} = \lambda \mathbf{v}\}$$

The eigenspace is a **subspace** of $\mathbb{R}^n$ (it is the null space of $A - \lambda I$).

The eigenvectors are the nonzero elements of $E_\lambda$.

---

## 4. The Eigenvalue Equation

$$A\mathbf{v} = \lambda \mathbf{v} \;\Leftrightarrow\; (A - \lambda I)\mathbf{v} = \mathbf{0}$$

This has a nonzero solution $\mathbf{v}$ iff:

$$\det(A - \lambda I) = 0$$

```
  The eigenvalue pipeline:
  
  ┌──────────────┐     ┌─────────────────────┐     ┌──────────────────┐
  │ Matrix A     │ --> │ Solve det(A−λI) = 0  │ --> │ Eigenvalues λ    │
  └──────────────┘     └─────────────────────┘     └──────────────────┘
                                                          │
                                                          ▼
                                                   ┌──────────────────┐
                                                   │ For each λ, solve│
                                                   │ (A−λI)v = 0      │
                                                   │ → Eigenvectors   │
                                                   └──────────────────┘
```

---

## 5. Examples

### Example 1: Diagonal Matrix

$$A = \begin{pmatrix} 3 & 0 \\ 0 & 7 \end{pmatrix}$$

The eigenvalues are the diagonal entries: $\lambda_1 = 3, \lambda_2 = 7$.

Eigenvectors: $\mathbf{v}_1 = (1, 0)^T$ for $\lambda_1 = 3$, $\mathbf{v}_2 = (0, 1)^T$ for $\lambda_2 = 7$.

**Rule:** For diagonal matrices, eigenvalues = diagonal entries, eigenvectors = standard basis vectors.

### Example 2: 2×2 Matrix

$$A = \begin{pmatrix} 4 & 1 \\ 2 & 3 \end{pmatrix}$$

$$\det(A - \lambda I) = (4-\lambda)(3-\lambda) - 2 = \lambda^2 - 7\lambda + 10 = (\lambda - 5)(\lambda - 2) = 0$$

$\lambda_1 = 5$: Solve $(A - 5I)\mathbf{v} = \mathbf{0}$

$$\begin{pmatrix} -1 & 1 \\ 2 & -2 \end{pmatrix} \mathbf{v} = \mathbf{0} \implies \mathbf{v}_1 = t\begin{pmatrix} 1 \\ 1 \end{pmatrix}$$

$\lambda_2 = 2$: Solve $(A - 2I)\mathbf{v} = \mathbf{0}$

$$\begin{pmatrix} 2 & 1 \\ 2 & 1 \end{pmatrix} \mathbf{v} = \mathbf{0} \implies \mathbf{v}_2 = t\begin{pmatrix} 1 \\ -2 \end{pmatrix}$$

### Example 3: Rotation Matrix (No Real Eigenvalues)

$$R_{90°} = \begin{pmatrix} 0 & -1 \\ 1 & 0 \end{pmatrix}$$

$$\det(R - \lambda I) = \lambda^2 + 1 = 0 \implies \lambda = \pm i$$

A 90° rotation has no real eigenvalues — no real vector keeps its direction under 90° rotation.

### Example 4: Projection Matrix

$$P = \begin{pmatrix} 1 & 0 \\ 0 & 0 \end{pmatrix} \quad \text{(projection onto } x\text{-axis)}$$

$\lambda_1 = 1$: vectors on the $x$-axis (they stay fixed)
$\lambda_2 = 0$: vectors on the $y$-axis (they map to $\mathbf{0}$)

---

## 6. Geometric Interpretation

| $\lambda$ value | Geometric effect |
|:-:|---|
| $\lambda > 1$ | Stretches eigenvector |
| $0 < \lambda < 1$ | Shrinks eigenvector |
| $\lambda = 1$ | Fixes eigenvector (invariant direction) |
| $\lambda = 0$ | Maps eigenvector to $\mathbf{0}$ (kernel direction) |
| $\lambda < 0$ | Reverses and scales eigenvector |
| $\lambda \in \mathbb{C}$ | Rotation (no invariant real direction) |

```
  λ > 1          0 < λ < 1        λ < 0           λ = 0
  
  ─────→         ──→               ←────           ·
  ════════════►   ═══►              ◄══════         (to origin)
  
  (stretches)    (shrinks)        (reverses)       (annihilates)
```

---

## 7. Eigenvalues of Special Matrices

| Matrix type | Eigenvalue property |
|------------|-------------------|
| Diagonal $\text{diag}(d_1, \dots, d_n)$ | $\lambda_i = d_i$ |
| Triangular | $\lambda_i =$ diagonal entries |
| Identity $I$ | All $\lambda_i = 1$ |
| Zero matrix | All $\lambda_i = 0$ |
| Idempotent ($A^2 = A$) | $\lambda \in \{0, 1\}$ |
| Nilpotent ($A^k = 0$) | All $\lambda_i = 0$ |
| Invertible $A$ | No $\lambda_i = 0$ |
| Orthogonal ($A^TA = I$) | $|\lambda| = 1$ |
| Symmetric ($A = A^T$) | All $\lambda_i \in \mathbb{R}$ |

---

## 8. Basic Properties

1. $\lambda$ is an eigenvalue of $A$ $\Leftrightarrow$ $\det(A - \lambda I) = 0$
2. If $\lambda$ is an eigenvalue of $A$, then $\lambda^k$ is an eigenvalue of $A^k$
3. If $A$ is invertible and $\lambda$ is an eigenvalue, then $\lambda^{-1}$ is an eigenvalue of $A^{-1}$
4. $\lambda$ is an eigenvalue of $A$ iff $\lambda + c$ is an eigenvalue of $A + cI$
5. $\text{tr}(A) = \sum \lambda_i$, $\det(A) = \prod \lambda_i$

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Eigenvalue | Scalar $\lambda$ s.t. $A\mathbf{v} = \lambda \mathbf{v}$ for some $\mathbf{v} \neq \mathbf{0}$ |
| Eigenvector | Nonzero vector satisfying $A\mathbf{v} = \lambda \mathbf{v}$ |
| Eigenspace $E_\lambda$ | $\ker(A - \lambda I)$, a subspace |
| Finding eigenvalues | Solve $\det(A - \lambda I) = 0$ |
| Finding eigenvectors | Solve $(A - \lambda I)\mathbf{v} = \mathbf{0}$ |
| Diagonal/Triangular | Eigenvalues = diagonal entries |
| Trace & Determinant | $\text{tr}(A) = \sum \lambda_i$, $\det(A) = \prod \lambda_i$ |

---

## Quick Revision Questions

1. Why must eigenvectors be nonzero by definition?

2. Find the eigenvalues and eigenvectors of $A = \begin{pmatrix} 2 & 1 \\ 0 & 3 \end{pmatrix}$.

3. A matrix has eigenvalue $\lambda = 0$. What does this tell you about the matrix?

4. Why does a rotation matrix by $90°$ have no real eigenvalues?

5. Prove: if $\lambda$ is an eigenvalue of $A$, then $\lambda^2$ is an eigenvalue of $A^2$.

---

[← Previous: Unit 3 — Orthonormal Basis](../03-Inner-Product-Spaces/06-orthonormal-basis.md) · [Back to README](../README.md) · [Next: Characteristic Polynomial →](02-characteristic-polynomial.md)
