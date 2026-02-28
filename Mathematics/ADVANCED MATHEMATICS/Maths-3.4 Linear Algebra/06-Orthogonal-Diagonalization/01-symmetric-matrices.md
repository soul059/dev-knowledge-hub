# 6.1 Symmetric Matrices

[← Previous: Unit 5 — Non-Diagonalizable Matrices](../05-Diagonalization/05-non-diagonalizable-matrices.md) · [Back to README](../README.md) · [Next: Spectral Theorem →](02-spectral-theorem.md)

---

## Chapter Overview

**Symmetric matrices** ($A = A^T$) are the most important class of matrices in applications. They arise naturally in physics, statistics, optimisation, and geometry. This chapter develops their remarkable eigenvalue properties, which set the stage for the spectral theorem.

---

## 1. Definition and Examples

> **Definition.** A real matrix $A$ is **symmetric** if $A = A^T$.

Equivalently: $a_{ij} = a_{ji}$ for all $i, j$.

### Common examples of symmetric matrices:

| Source | Matrix |
|--------|--------|
| Distance/adjacency | $D_{ij} = D_{ji}$ (symmetric by nature) |
| Covariance matrix | $\Sigma = \frac{1}{n}X^TX$ |
| Gram matrix | $G = A^TA$ |
| Hessian matrix | $H_{ij} = \frac{\partial^2 f}{\partial x_i \partial x_j}$ |
| Moment of inertia | $I_{ij}$ in mechanics |
| Laplacian matrix | Graph theory |

```
  Symmetric matrix structure:
  
  ┌─────────────────────────┐
  │  a   b   c   d          │
  │  b   e   f   g    A = Aᵀ│
  │  c   f   h   i          │
  │  d   g   i   j          │
  └─────────────────────────┘
  
  Only n(n+1)/2 independent entries (upper triangle determines all)
```

---

## 2. Fundamental Properties of Symmetric Matrices

> **Theorem.** If $A$ is a real symmetric matrix, then:
>
> 1. **All eigenvalues are real**
> 2. **Eigenvectors from distinct eigenvalues are orthogonal**
> 3. **$A$ is always diagonalisable** ($\text{gm} = \text{am}$ for every eigenvalue)
> 4. **$A$ is orthogonally diagonalisable:** $A = QDQ^T$ with $Q$ orthogonal

These are the Four Pillars that make symmetric matrices special.

---

## 3. Proof: Eigenvalues Are Real

Let $A\mathbf{v} = \lambda \mathbf{v}$ with $\mathbf{v} \neq \mathbf{0}$ (possibly complex).

$$\lambda \overline{\mathbf{v}}^T \mathbf{v} = \overline{\mathbf{v}}^T (A\mathbf{v}) = \overline{\mathbf{v}}^T A \mathbf{v}$$

Taking complex conjugate transpose: since $A = A^T$ is real,

$$\overline{\overline{\mathbf{v}}^T A \mathbf{v}} = \mathbf{v}^T A^T \overline{\mathbf{v}} = \mathbf{v}^T A \overline{\mathbf{v}} = (A\mathbf{v})^T \overline{\mathbf{v}} = (\lambda \mathbf{v})^T \overline{\mathbf{v}} = \lambda \overline{\mathbf{v}}^T \mathbf{v}$$

Wait — we also get $\overline{\lambda} \cdot \overline{\mathbf{v}}^T \mathbf{v}$. So:

$$\lambda \|\mathbf{v}\|^2 = \overline{\lambda} \|\mathbf{v}\|^2 \implies \lambda = \overline{\lambda} \implies \lambda \in \mathbb{R} \quad \blacksquare$$

---

## 4. Proof: Eigenvectors Are Orthogonal (Distinct Eigenvalues)

Let $A\mathbf{v}_1 = \lambda_1 \mathbf{v}_1$ and $A\mathbf{v}_2 = \lambda_2 \mathbf{v}_2$ with $\lambda_1 \neq \lambda_2$.

$$\lambda_1 \langle \mathbf{v}_1, \mathbf{v}_2 \rangle = \langle A\mathbf{v}_1, \mathbf{v}_2 \rangle = \langle \mathbf{v}_1, A^T\mathbf{v}_2 \rangle = \langle \mathbf{v}_1, A\mathbf{v}_2 \rangle = \lambda_2 \langle \mathbf{v}_1, \mathbf{v}_2 \rangle$$

$$(\lambda_1 - \lambda_2) \langle \mathbf{v}_1, \mathbf{v}_2 \rangle = 0$$

Since $\lambda_1 \neq \lambda_2$: $\langle \mathbf{v}_1, \mathbf{v}_2 \rangle = 0$ ∎

---

## 5. Symmetric vs General Matrices

| Property | General $A$ | Symmetric $A = A^T$ |
|----------|------------|---------------------|
| Eigenvalues | May be complex | Always real |
| Eigenvectors | Linearly independent (if diag.) | Orthogonal |
| Diagonalisable? | Not always | **Always** |
| Diagonalisation | $A = PDP^{-1}$ | $A = QDQ^T$ ($Q$ orthogonal!) |
| Need $P^{-1}$? | Yes (may be expensive) | No! $Q^{-1} = Q^T$ (free!) |

---

## 6. Worked Example

$$A = \begin{pmatrix} 2 & 1 \\ 1 & 2 \end{pmatrix}$$

$p(\lambda) = \lambda^2 - 4\lambda + 3 = (\lambda - 1)(\lambda - 3)$

$\lambda_1 = 1$: $\mathbf{v}_1 = \begin{pmatrix} 1 \\ -1 \end{pmatrix}$, normalise: $\mathbf{q}_1 = \frac{1}{\sqrt{2}}\begin{pmatrix} 1 \\ -1 \end{pmatrix}$

$\lambda_2 = 3$: $\mathbf{v}_2 = \begin{pmatrix} 1 \\ 1 \end{pmatrix}$, normalise: $\mathbf{q}_2 = \frac{1}{\sqrt{2}}\begin{pmatrix} 1 \\ 1 \end{pmatrix}$

**Check orthogonality:** $\mathbf{q}_1 \cdot \mathbf{q}_2 = \frac{1}{2}(1 - 1) = 0$ ✓

$$Q = \frac{1}{\sqrt{2}}\begin{pmatrix} 1 & 1 \\ -1 & 1 \end{pmatrix}, \quad A = Q\begin{pmatrix} 1 & 0 \\ 0 & 3 \end{pmatrix}Q^T$$

and $Q^{-1} = Q^T$ — no matrix inversion needed!

---

## 7. Positive Definite Symmetric Matrices

> **Definition.** A symmetric matrix $A$ is **positive definite** if:
>
> $$\mathbf{x}^T A \mathbf{x} > 0 \quad \text{for all } \mathbf{x} \neq \mathbf{0}$$

| Type | Eigenvalue condition | Notation |
|------|---------------------|----------|
| Positive definite | All $\lambda_i > 0$ | $A \succ 0$ |
| Positive semi-definite | All $\lambda_i \geq 0$ | $A \succeq 0$ |
| Negative definite | All $\lambda_i < 0$ | $A \prec 0$ |
| Indefinite | Mixed signs | — |

**Tests for positive definiteness** (all equivalent):
1. All eigenvalues > 0
2. All leading principal minors > 0 (Sylvester's criterion)
3. All pivots in elimination > 0
4. $A = R^T R$ for some invertible $R$ (Cholesky)

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Symmetric matrix | $A = A^T$, $a_{ij} = a_{ji}$ |
| Real eigenvalues | Always guaranteed for symmetric $A$ |
| Orthogonal eigenvectors | Distinct eigenvalues → orthogonal |
| Always diagonalisable | $\text{gm} = \text{am}$ guaranteed |
| Orthogonal diagonalisation | $A = QDQ^T$ with $Q^{-1} = Q^T$ |
| Positive definite | All $\lambda > 0$ iff $\mathbf{x}^TA\mathbf{x} > 0$ for $\mathbf{x} \neq 0$ |
| Sylvester's criterion | Leading minors > 0 ↔ positive definite |

---

## Quick Revision Questions

1. Prove that eigenvalues of a real symmetric matrix are real.

2. Show that $A^TA$ is always symmetric and positive semi-definite.

3. Orthogonally diagonalise $A = \begin{pmatrix} 3 & 1 \\ 1 & 3 \end{pmatrix}$.

4. Is $A = \begin{pmatrix} 2 & -1 \\ -1 & 2 \end{pmatrix}$ positive definite? Use two different methods.

5. Why is orthogonal diagonalisation computationally superior to general diagonalisation?

---

[← Previous: Unit 5 — Non-Diagonalizable Matrices](../05-Diagonalization/05-non-diagonalizable-matrices.md) · [Back to README](../README.md) · [Next: Spectral Theorem →](02-spectral-theorem.md)
