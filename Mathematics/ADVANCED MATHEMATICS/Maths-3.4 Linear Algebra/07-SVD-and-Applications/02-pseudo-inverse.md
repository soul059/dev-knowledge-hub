# 7.2 The Pseudo-Inverse (Moore-Penrose Inverse)

[← Previous: Singular Value Decomposition](01-singular-value-decomposition.md) · [Back to README](../README.md) · [Next: Least Squares Problems →](03-least-squares-problems.md)

---

## Chapter Overview

Not every matrix has an inverse. The **pseudo-inverse** $A^+$ generalises the notion of invertibility to **all** matrices — rectangular, singular, or otherwise. It provides the "best possible" inverse using the SVD.

---

## 1. Motivation

For a square invertible matrix: $A^{-1}$ solves $A\mathbf{x} = \mathbf{b}$.

But what if:
- $A$ is rectangular ($m \neq n$)?
- $A$ is singular ($\det A = 0$)?
- $A\mathbf{x} = \mathbf{b}$ has no solution?

We need a generalised inverse that gives the "best" answer in all cases.

---

## 2. Definition via SVD

> If $A = U\Sigma V^T$ is the SVD of $A$ ($m \times n$, rank $r$), then the **Moore-Penrose pseudo-inverse** is:
>
> $$A^+ = V \Sigma^+ U^T$$
>
> where $\Sigma^+$ is the $n \times m$ matrix obtained by transposing $\Sigma$ and inverting the nonzero entries:
>
> $$\Sigma = \text{diag}(\sigma_1, \dots, \sigma_r, 0, \dots, 0)_{m \times n} \implies \Sigma^+ = \text{diag}(\sigma_1^{-1}, \dots, \sigma_r^{-1}, 0, \dots, 0)_{n \times m}$$

```
  Construction of A⁺:
  
  A = U Σ Vᵀ           →        A⁺ = V Σ⁺ Uᵀ
  (m×n)                          (n×m)
  
  Σ = ┌─────────┐              Σ⁺ = ┌─────────┐
      │1/σ₁     │ m                  │σ₁      │ n
      │  1/σ₂   │                    │  σ₂    │
      │    ·    │                    │    ·   │
      │     0   │                    │     0  │
      └─────────┘                    └─────────┘
           n                              m
  
  (Transpose and invert nonzero entries)
```

---

## 3. The Four Penrose Conditions

$A^+$ is the unique matrix satisfying all four:

| Condition | Formula |
|:---------:|---------|
| (1) | $AA^+A = A$ |
| (2) | $A^+AA^+ = A^+$ |
| (3) | $(AA^+)^T = AA^+$ |
| (4) | $(A^+A)^T = A^+A$ |

Conditions (3) and (4) say that $AA^+$ and $A^+A$ are symmetric (and they are projection matrices).

---

## 4. Properties

| Property | Formula |
|----------|---------|
| Size | $A^+$ is $n \times m$ (if $A$ is $m \times n$) |
| Invertible $A$ | $A^+ = A^{-1}$ |
| Zero matrix | $0^+ = 0$ |
| $(A^+)^+ = A$ | Involution |
| $(A^T)^+ = (A^+)^T$ | Transpose commutes |
| $(cA)^+ = c^{-1}A^+$ | Scalar (for $c \neq 0$) |
| rank$(A^+)$ = rank$(A)$ | Same rank |

---

## 5. Geometric Interpretation

$AA^+$ is the **orthogonal projection** onto $\text{Col}(A)$.

$A^+A$ is the **orthogonal projection** onto $\text{Row}(A)$.

```
  ℝⁿ ──────A──────> ℝᵐ
  │                   │
  │ A⁺A               │ AA⁺
  │ (proj onto         │ (proj onto
  │  Row(A))           │  Col(A))
  ▼                   ▼
  Row(A) ────A────> Col(A)
         <───A⁺────
```

---

## 6. Worked Example

Find $A^+$ for $A = \begin{pmatrix} 1 & 0 \\ 0 & 1 \\ 0 & 0 \end{pmatrix}$.

$A^TA = \begin{pmatrix} 1 & 0 \\ 0 & 1 \end{pmatrix} = I_2$

$\sigma_1 = 1, \sigma_2 = 1$, $V = I_2$

$\mathbf{u}_1 = A\mathbf{v}_1 / \sigma_1 = (1, 0, 0)^T$, $\mathbf{u}_2 = (0, 1, 0)^T$, extend: $\mathbf{u}_3 = (0, 0, 1)^T$

$$\Sigma = \begin{pmatrix} 1 & 0 \\ 0 & 1 \\ 0 & 0 \end{pmatrix}, \quad \Sigma^+ = \begin{pmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \end{pmatrix}$$

$$A^+ = V\Sigma^+ U^T = I_2 \begin{pmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \end{pmatrix} I_3 = \begin{pmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \end{pmatrix}$$

This is just $A^T$! (Because $A$ has orthonormal columns.)

**Verify Penrose conditions:** $AA^+A = \begin{pmatrix} 1 & 0 \\ 0 & 1 \\ 0 & 0 \end{pmatrix}\begin{pmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \end{pmatrix}\begin{pmatrix} 1 & 0 \\ 0 & 1 \\ 0 & 0 \end{pmatrix} = \begin{pmatrix} 1 & 0 \\ 0 & 1 \\ 0 & 0 \end{pmatrix} = A$ ✓

---

## 7. Special Cases

| Matrix type | Pseudo-inverse |
|---|---|
| Invertible ($m = n$, full rank) | $A^+ = A^{-1}$ |
| Full column rank ($m > n$, rank $n$) | $A^+ = (A^TA)^{-1}A^T$ (left inverse) |
| Full row rank ($m < n$, rank $m$) | $A^+ = A^T(AA^T)^{-1}$ (right inverse) |
| Orthogonal columns | $A^+ = A^T$ (when normalised) |
| Rank 1: $A = \mathbf{u}\sigma\mathbf{v}^T$ | $A^+ = \mathbf{v}\sigma^{-1}\mathbf{u}^T$ |

---

## 8. The Pseudo-Inverse Solution

For $A\mathbf{x} = \mathbf{b}$:

$$\mathbf{x}^* = A^+\mathbf{b}$$

This gives the **minimum-norm least-squares** solution:
- Among all $\mathbf{x}$ minimising $\|A\mathbf{x} - \mathbf{b}\|$, it picks the one with smallest $\|\mathbf{x}\|$.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Pseudo-inverse $A^+$ | $V\Sigma^+U^T$ from SVD |
| Size | $n \times m$ for $m \times n$ matrix |
| Penrose conditions | 4 conditions uniquely define $A^+$ |
| Invertible case | $A^+ = A^{-1}$ |
| $AA^+$ | Projection onto Col$(A)$ |
| $A^+A$ | Projection onto Row$(A)$ |
| $A^+\mathbf{b}$ | Minimum-norm least-squares solution |

---

## Quick Revision Questions

1. Find the pseudo-inverse of $A = \begin{pmatrix} 3 & 0 \\ 0 & 0 \end{pmatrix}$.

2. State the four Penrose conditions. Why do they uniquely determine $A^+$?

3. If $A$ has full column rank, show $A^+ = (A^TA)^{-1}A^T$.

4. What are $AA^+$ and $A^+A$ geometrically?

5. Find $A^+$ for $A = (1 \quad 2 \quad 2)$ (a row vector).

---

[← Previous: Singular Value Decomposition](01-singular-value-decomposition.md) · [Back to README](../README.md) · [Next: Least Squares Problems →](03-least-squares-problems.md)
