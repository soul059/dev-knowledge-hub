# 4.4 Properties of Eigenvalues and Eigenvectors

[← Previous: Finding Eigenvalues & Eigenvectors](03-finding-eigenvalues-eigenvectors.md) · [Back to README](../README.md) · [Next: Algebraic & Geometric Multiplicity →](05-algebraic-geometric-multiplicity.md)

---

## Chapter Overview

This chapter collects the essential theorems relating eigenvalues to matrix properties — invertibility, powers, transposes, and linear independence of eigenvectors.

---

## 1. Eigenvalues Under Matrix Operations

If $\lambda$ is an eigenvalue of $A$ with eigenvector $\mathbf{v}$:

| Operation on $A$ | Eigenvalue | Eigenvector |
|:---:|:---:|:---:|
| $A$ | $\lambda$ | $\mathbf{v}$ |
| $A^k$ | $\lambda^k$ | $\mathbf{v}$ |
| $A^{-1}$ (if exists) | $\lambda^{-1}$ | $\mathbf{v}$ |
| $A + cI$ | $\lambda + c$ | $\mathbf{v}$ |
| $cA$ | $c\lambda$ | $\mathbf{v}$ |
| $A^T$ | $\lambda$ | (different $\mathbf{v}$ in general) |
| $p(A)$ for polynomial $p$ | $p(\lambda)$ | $\mathbf{v}$ |

**Proof for $A^k$:** By induction. $A\mathbf{v} = \lambda \mathbf{v} \implies A^2 \mathbf{v} = A(A\mathbf{v}) = A(\lambda \mathbf{v}) = \lambda A\mathbf{v} = \lambda^2 \mathbf{v}$ ∎

**Proof for $A^{-1}$:** $A\mathbf{v} = \lambda \mathbf{v} \implies \mathbf{v} = A^{-1}(\lambda \mathbf{v}) = \lambda A^{-1}\mathbf{v} \implies A^{-1}\mathbf{v} = \frac{1}{\lambda}\mathbf{v}$ ∎

---

## 2. Eigenvalues and Invertibility

> **Theorem.** $A$ is singular (not invertible) if and only if $\lambda = 0$ is an eigenvalue of $A$.

**Proof:** $0$ is an eigenvalue $\Leftrightarrow$ $\det(A - 0 \cdot I) = 0$ $\Leftrightarrow$ $\det(A) = 0$ $\Leftrightarrow$ $A$ is singular. ∎

This connects to the **Invertible Matrix Theorem** — adding another equivalent condition:

```
  A is invertible
       ⟺ det(A) ≠ 0
       ⟺ λ = 0 is NOT an eigenvalue
       ⟺ ker(A) = {0}
       ⟺ A has full rank
       ⟺ columns are linearly independent
       ⟺ ...
```

---

## 3. Linear Independence of Eigenvectors

> **Theorem.** Eigenvectors corresponding to **distinct** eigenvalues are linearly independent.

**Proof (for 2 eigenvectors):**

Let $A\mathbf{v}_1 = \lambda_1 \mathbf{v}_1$ and $A\mathbf{v}_2 = \lambda_2 \mathbf{v}_2$ with $\lambda_1 \neq \lambda_2$.

Suppose $c_1 \mathbf{v}_1 + c_2 \mathbf{v}_2 = \mathbf{0}$ ... (★)

Multiply (★) by $A$: $c_1 \lambda_1 \mathbf{v}_1 + c_2 \lambda_2 \mathbf{v}_2 = \mathbf{0}$ ... (★★)

Compute $\lambda_1 \times (★) - (★★)$: $c_2(\lambda_1 - \lambda_2)\mathbf{v}_2 = \mathbf{0}$

Since $\lambda_1 \neq \lambda_2$ and $\mathbf{v}_2 \neq \mathbf{0}$: $c_2 = 0$. Similarly $c_1 = 0$. ∎

> **Corollary.** An $n \times n$ matrix with $n$ distinct eigenvalues has $n$ linearly independent eigenvectors.

---

## 4. Eigenvalues of Symmetric Matrices

> **Theorem.** If $A = A^T$ (real symmetric), then:
> 1. All eigenvalues are **real**
> 2. Eigenvectors from different eigenvalues are **orthogonal**

**Proof of (1):** Let $A\mathbf{v} = \lambda \mathbf{v}$. Then:
$$\lambda \|\mathbf{v}\|^2 = \lambda \bar{\mathbf{v}}^T \mathbf{v} = \bar{\mathbf{v}}^T A\mathbf{v} = (A\bar{\mathbf{v}})^T \mathbf{v} = (\bar{\lambda}\bar{\mathbf{v}})^T \mathbf{v} = \bar{\lambda} \|\mathbf{v}\|^2$$

Since $\|\mathbf{v}\|^2 \neq 0$: $\lambda = \bar{\lambda}$, so $\lambda \in \mathbb{R}$. ∎

**Proof of (2):** Let $A\mathbf{v}_1 = \lambda_1 \mathbf{v}_1$, $A\mathbf{v}_2 = \lambda_2 \mathbf{v}_2$, $\lambda_1 \neq \lambda_2$.

$$\lambda_1 \langle \mathbf{v}_1, \mathbf{v}_2 \rangle = \langle A\mathbf{v}_1, \mathbf{v}_2 \rangle = \langle \mathbf{v}_1, A^T\mathbf{v}_2 \rangle = \langle \mathbf{v}_1, A\mathbf{v}_2 \rangle = \lambda_2 \langle \mathbf{v}_1, \mathbf{v}_2 \rangle$$

$(\lambda_1 - \lambda_2)\langle \mathbf{v}_1, \mathbf{v}_2 \rangle = 0 \implies \langle \mathbf{v}_1, \mathbf{v}_2 \rangle = 0$ ∎

---

## 5. Eigenvalues of Special Matrices

| Matrix Property | Eigenvalue Property | Eigenvector Property |
|---|---|---|
| Symmetric ($A = A^T$) | All real | Orthogonal for distinct $\lambda$ |
| Skew-symmetric ($A = -A^T$) | Pure imaginary or zero | — |
| Orthogonal ($A^TA = I$) | $|\lambda| = 1$ | — |
| Idempotent ($A^2 = A$) | $\lambda \in \{0, 1\}$ | — |
| Nilpotent ($A^k = 0$) | All $\lambda = 0$ | — |
| Positive definite | All $\lambda > 0$ | — |
| Positive semi-definite | All $\lambda \geq 0$ | — |

**Proof for idempotent:** $A\mathbf{v} = \lambda \mathbf{v} \implies A^2\mathbf{v} = \lambda^2 \mathbf{v}$. But $A^2 = A$, so $\lambda \mathbf{v} = \lambda^2 \mathbf{v} \implies \lambda = \lambda^2 \implies \lambda \in \{0, 1\}$. ∎

---

## 6. Spectrum and Spectral Radius

> **Definition.** The **spectrum** of $A$ is the set of all eigenvalues:
>
> $$\sigma(A) = \{\lambda_1, \lambda_2, \dots, \lambda_k\}$$

> **Definition.** The **spectral radius** of $A$ is:
>
> $$\rho(A) = \max\{|\lambda| : \lambda \in \sigma(A)\}$$

The spectral radius controls:
- **Convergence of $A^k$:** $A^k \to 0$ iff $\rho(A) < 1$
- **Convergence of iterative methods:** $(I - A)^{-1} = I + A + A^2 + \dots$ converges iff $\rho(A) < 1$

---

## 7. Eigenvalue Decomposition Preview

If $A$ has $n$ linearly independent eigenvectors $\mathbf{v}_1, \dots, \mathbf{v}_n$:

$$A = PDP^{-1}$$

where $P = [\mathbf{v}_1 | \cdots | \mathbf{v}_n]$ and $D = \text{diag}(\lambda_1, \dots, \lambda_n)$.

This is the subject of Unit 5 (Diagonalization).

```
  A = P D P⁻¹
  
  ┌───┐   ┌───┐ ┌───────┐ ┌─────┐
  │ A │ = │ P │ │ λ₁    │ │P⁻¹  │
  │   │   │   │ │  λ₂   │ │     │
  │   │   │   │ │   λ₃  │ │     │
  └───┘   └───┘ └───────┘ └─────┘
  
  Change    Scale    Change
  to eig.   by λ     back
  basis
```

---

## Summary Table

| Property | Statement |
|----------|-----------|
| Powers | $\lambda$ of $A$ $\implies$ $\lambda^k$ of $A^k$ |
| Inverse | $\lambda$ of $A$ $\implies$ $1/\lambda$ of $A^{-1}$ |
| Shift | $\lambda$ of $A$ $\implies$ $\lambda + c$ of $A + cI$ |
| Transpose | Same eigenvalues (different eigenvectors) |
| Distinct eigenvalues | Eigenvectors are linearly independent |
| Symmetric matrix | Real eigenvalues, orthogonal eigenvectors |
| Singular matrix | Has eigenvalue $\lambda = 0$ |
| Spectral radius | $\rho(A) = \max|\lambda_i|$, controls convergence |

---

## Quick Revision Questions

1. If $A$ has eigenvalues $2, 3, 5$, find the eigenvalues of $A^2 - 3A + I$.

2. Prove that a nilpotent matrix has all eigenvalues equal to zero.

3. If $A$ is orthogonal and $\lambda$ is a real eigenvalue, show $\lambda = \pm 1$.

4. Are eigenvectors of $A$ and $A^T$ always the same? Prove or give a counterexample.

5. Show that if $\mathbf{v}_1, \mathbf{v}_2$ are eigenvectors for the same eigenvalue $\lambda$, then $c_1\mathbf{v}_1 + c_2\mathbf{v}_2$ is also an eigenvector (if nonzero).

---

[← Previous: Finding Eigenvalues & Eigenvectors](03-finding-eigenvalues-eigenvectors.md) · [Back to README](../README.md) · [Next: Algebraic & Geometric Multiplicity →](05-algebraic-geometric-multiplicity.md)
