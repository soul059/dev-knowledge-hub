# 7.1 Singular Value Decomposition (SVD)

[← Previous: Unit 6 — Quadratic Forms](../06-Orthogonal-Diagonalization/04-quadratic-forms.md) · [Back to README](../README.md) · [Next: Pseudo-Inverse →](02-pseudo-inverse.md)

---

## Chapter Overview

The **Singular Value Decomposition** generalises eigenvalue decomposition to **any** matrix — even rectangular ones. It is arguably the single most important matrix factorisation in applied mathematics, underpinning everything from data science to signal processing.

---

## 1. Motivation

Eigenvalue decomposition requires:
- Square matrix
- Enough eigenvectors (diagonalisable)

The SVD works for **any** $m \times n$ matrix, even when $m \neq n$.

---

## 2. The SVD Theorem

> **Theorem.** Every $m \times n$ matrix $A$ (real or complex) can be factored as:
>
> $$A = U \Sigma V^T$$
>
> where:
> - $U$ is $m \times m$ orthogonal (left singular vectors)
> - $\Sigma$ is $m \times n$ "diagonal" with non-negative entries $\sigma_1 \geq \sigma_2 \geq \dots \geq \sigma_r > 0$ (singular values)
> - $V$ is $n \times n$ orthogonal (right singular vectors)
> - $r = \text{rank}(A)$

```
  A = U Σ Vᵀ
  
  ┌─────┐   ┌─────┐ ┌───────┐ ┌─────┐ᵀ
  │     │   │     │ │σ₁     │ │     │
  │  A  │ = │  U  │ │ σ₂    │ │  V  │
  │     │   │     │ │  ·    │ │     │
  │m × n│   │m × m│ │   σᵣ  │ │n × n│
  │     │   │     │ │    0  │ │     │
  └─────┘   └─────┘ └───────┘ └─────┘
              ↑       m × n      ↑
          orthogonal          orthogonal
```

---

## 3. Understanding the Components

| Component | Size | Properties | Interpretation |
|-----------|------|------------|---------------|
| $U = [\mathbf{u}_1 \cdots \mathbf{u}_m]$ | $m \times m$ | $U^TU = I$ | ONB for $\mathbb{R}^m$ (output space) |
| $\Sigma$ | $m \times n$ | $\sigma_1 \geq \dots \geq \sigma_r > 0$, rest zero | Scaling factors |
| $V = [\mathbf{v}_1 \cdots \mathbf{v}_n]$ | $n \times n$ | $V^TV = I$ | ONB for $\mathbb{R}^n$ (input space) |

The action of $A$ on $\mathbf{v}_i$:

$$A\mathbf{v}_i = \sigma_i \mathbf{u}_i \quad (i = 1, \dots, r)$$
$$A\mathbf{v}_i = \mathbf{0} \quad (i = r+1, \dots, n)$$

---

## 4. Computing the SVD

**Step 1:** Compute $A^TA$ (symmetric $n \times n$). Find its eigenvalues $\lambda_1 \geq \dots \geq \lambda_n \geq 0$.

**Step 2:** Singular values: $\sigma_i = \sqrt{\lambda_i}$ for $i = 1, \dots, r$ (where $\lambda_i > 0$).

**Step 3:** Right singular vectors $V$: orthonormal eigenvectors of $A^TA$.

**Step 4:** Left singular vectors: $\mathbf{u}_i = \frac{1}{\sigma_i}A\mathbf{v}_i$ for $i = 1, \dots, r$.

**Step 5:** Extend $\{\mathbf{u}_1, \dots, \mathbf{u}_r\}$ to an ONB of $\mathbb{R}^m$.

```
  SVD computation pipeline:
  
  ┌─────┐     ┌─────────┐     ┌──────────────┐     ┌───────────┐
  │  A  │ --> │ AᵀA     │ --> │ Eigenvalues  │ --> │ σᵢ = √λᵢ  │
  └─────┘     │(symm.)  │     │ & eigenvecs  │     │ & V       │
              └─────────┘     └──────────────┘     └─────┬─────┘
                                                         │
                                                         ▼
                                                  ┌──────────────┐
                                                  │ uᵢ = Avᵢ/σᵢ │
                                                  │ → U           │
                                                  └──────────────┘
```

---

## 5. Worked Example

Find the SVD of $A = \begin{pmatrix} 4 & 0 \\ 3 & -5 \end{pmatrix}$.

**Step 1:** $A^TA = \begin{pmatrix} 4 & 3 \\ 0 & -5 \end{pmatrix}\begin{pmatrix} 4 & 0 \\ 3 & -5 \end{pmatrix} = \begin{pmatrix} 25 & -15 \\ -15 & 25 \end{pmatrix}$

**Step 2:** $p(\lambda) = \lambda^2 - 50\lambda + 400 = (\lambda - 10)(\lambda - 40)$

$\lambda_1 = 40, \lambda_2 = 10 \implies \sigma_1 = \sqrt{40} = 2\sqrt{10}, \sigma_2 = \sqrt{10}$

**Step 3:** Eigenvectors of $A^TA$:

$\lambda = 40$: $\mathbf{v}_1 = \frac{1}{\sqrt{2}}(1, -1)^T$

$\lambda = 10$: $\mathbf{v}_2 = \frac{1}{\sqrt{2}}(1, 1)^T$

**Step 4:**

$\mathbf{u}_1 = \frac{1}{\sigma_1} A\mathbf{v}_1 = \frac{1}{2\sqrt{10}} \begin{pmatrix} 4 & 0 \\ 3 & -5 \end{pmatrix} \frac{1}{\sqrt{2}}\begin{pmatrix} 1 \\ -1 \end{pmatrix} = \frac{1}{2\sqrt{20}}\begin{pmatrix} 4 \\ 8 \end{pmatrix} = \frac{1}{\sqrt{5}}\begin{pmatrix} 1 \\ 2 \end{pmatrix}$

$\mathbf{u}_2 = \frac{1}{\sigma_2} A\mathbf{v}_2 = \frac{1}{\sqrt{10}} \begin{pmatrix} 4 & 0 \\ 3 & -5 \end{pmatrix} \frac{1}{\sqrt{2}}\begin{pmatrix} 1 \\ 1 \end{pmatrix} = \frac{1}{\sqrt{20}}\begin{pmatrix} 4 \\ -2 \end{pmatrix} = \frac{1}{\sqrt{5}}\begin{pmatrix} 2 \\ -1 \end{pmatrix}$

**Result:**

$$A = \frac{1}{\sqrt{5}}\begin{pmatrix} 1 & 2 \\ 2 & -1 \end{pmatrix} \begin{pmatrix} 2\sqrt{10} & 0 \\ 0 & \sqrt{10} \end{pmatrix} \frac{1}{\sqrt{2}}\begin{pmatrix} 1 & 1 \\ -1 & 1 \end{pmatrix}$$

---

## 6. Geometric Interpretation

The SVD reveals what $A$ does geometrically:

$$A = U \Sigma V^T: \quad \text{Rotate}(V^T) \to \text{Scale}(\Sigma) \to \text{Rotate}(U)$$

Every linear map is a rotation, followed by scaling along coordinate axes, followed by another rotation.

```
  Action of A on the unit circle:
  
  Unit circle ──Vᵀ──> Rotated circle ──Σ──> Ellipse ──U──> Rotated ellipse
  
      ○          →         ○         →       ⬭       →        ⬭
    (round)            (round)          (stretched)      (stretched
                                         along axes)      and rotated)
                                         
  Semi-axes of the ellipse = σ₁ and σ₂
```

---

## 7. SVD and the Four Fundamental Subspaces

| Subspace | Dimension | Basis from SVD |
|----------|:---------:|----------------|
| Column space $\text{Col}(A)$ | $r$ | $\{\mathbf{u}_1, \dots, \mathbf{u}_r\}$ |
| Left null space $\text{Null}(A^T)$ | $m - r$ | $\{\mathbf{u}_{r+1}, \dots, \mathbf{u}_m\}$ |
| Row space $\text{Row}(A)$ | $r$ | $\{\mathbf{v}_1, \dots, \mathbf{v}_r\}$ |
| Null space $\text{Null}(A)$ | $n - r$ | $\{\mathbf{v}_{r+1}, \dots, \mathbf{v}_n\}$ |

---

## 8. Key Facts

| Fact | Result |
|------|--------|
| $\text{rank}(A)$ | Number of nonzero singular values |
| $\|A\|_2$ (operator norm) | $\sigma_1$ (largest singular value) |
| $\|A\|_F$ (Frobenius norm) | $\sqrt{\sigma_1^2 + \dots + \sigma_r^2}$ |
| $\text{cond}(A)$ (condition number) | $\sigma_1 / \sigma_r$ |
| $\det(A)$ (if square) | $\pm \prod \sigma_i$ |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| SVD | $A = U\Sigma V^T$, works for **any** matrix |
| Singular values | $\sigma_i = \sqrt{\lambda_i(A^TA)}$, ordered $\sigma_1 \geq \dots \geq \sigma_r > 0$ |
| $U$ | Left singular vectors (ONB of $\mathbb{R}^m$) |
| $V$ | Right singular vectors (ONB of $\mathbb{R}^n$) |
| Geometry | Rotation → Scale → Rotation |
| Rank | Number of nonzero $\sigma_i$ |
| Computation | Eigendecompose $A^TA$ → $V, \sigma_i$ → $U$ |

---

## Quick Revision Questions

1. State the SVD theorem. What are the dimensions and properties of $U$, $\Sigma$, $V$?

2. How do singular values relate to eigenvalues of $A^TA$?

3. Find the SVD of $A = \begin{pmatrix} 1 & 1 \\ 0 & 0 \end{pmatrix}$.

4. What is the geometric interpretation of the SVD?

5. How can you read the rank of $A$ from its SVD?

---

[← Previous: Unit 6 — Quadratic Forms](../06-Orthogonal-Diagonalization/04-quadratic-forms.md) · [Back to README](../README.md) · [Next: Pseudo-Inverse →](02-pseudo-inverse.md)
