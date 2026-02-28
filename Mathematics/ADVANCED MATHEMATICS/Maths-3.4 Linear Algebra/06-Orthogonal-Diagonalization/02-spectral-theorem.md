# 6.2 The Spectral Theorem

[← Previous: Symmetric Matrices](01-symmetric-matrices.md) · [Back to README](../README.md) · [Next: Orthogonal Matrices →](03-orthogonal-matrices.md)

---

## Chapter Overview

The **Spectral Theorem** is arguably the most important theorem in linear algebra. It guarantees that every real symmetric matrix can be **orthogonally diagonalised** — decomposed into a sum of rank-1 projection matrices weighted by eigenvalues.

---

## 1. Statement (Real Version)

> **Spectral Theorem (Real).** Let $A$ be an $n \times n$ real symmetric matrix. Then:
>
> 1. $A$ has $n$ real eigenvalues (counted with multiplicity)
> 2. There exists an orthogonal matrix $Q$ ($Q^TQ = I$) such that $A = QDQ^T$
> 3. The columns of $Q$ form an orthonormal eigenbasis for $\mathbb{R}^n$

---

## 2. The Spectral Decomposition

The factorisation $A = QDQ^T$ can be expanded as:

$$A = \lambda_1 \mathbf{q}_1 \mathbf{q}_1^T + \lambda_2 \mathbf{q}_2 \mathbf{q}_2^T + \dots + \lambda_n \mathbf{q}_n \mathbf{q}_n^T$$

Each $P_i = \mathbf{q}_i \mathbf{q}_i^T$ is a **rank-1 orthogonal projection** onto the eigenspace.

```
  Spectral Decomposition:
  
  A = λ₁·q₁q₁ᵀ + λ₂·q₂q₂ᵀ + ··· + λₙ·qₙqₙᵀ
      ────┬────   ────┬────         ────┬────
      Scale by    Scale by          Scale by
      λ₁ along   λ₂ along          λₙ along
      direction  direction         direction
      q₁         q₂                qₙ
  
  Each qᵢqᵢᵀ is a projection matrix onto span{qᵢ}
```

### Properties of the Projection Matrices $P_i = \mathbf{q}_i\mathbf{q}_i^T$:

| Property | Result |
|----------|--------|
| $P_i^2 = P_i$ | Idempotent (projection) |
| $P_i^T = P_i$ | Symmetric |
| $P_i P_j = 0$ ($i \neq j$) | Mutually orthogonal |
| $\sum P_i = I$ | Resolution of identity |
| rank($P_i$) = 1 | Rank-1 projections |

---

## 3. Orthogonal Diagonalisation Procedure

```
  ┌──────────────────────────────────────────┐
  │  ORTHOGONAL DIAGONALISATION ALGORITHM     │
  ├──────────────────────────────────────────┤
  │  1. Verify A = Aᵀ                        │
  │  2. Find eigenvalues λ₁, ..., λₖ         │
  │  3. Find basis for each eigenspace E_λᵢ  │
  │  4. Apply Gram-Schmidt WITHIN each        │
  │     eigenspace (if gm > 1)               │
  │  5. Normalise all eigenvectors            │
  │  6. Form Q = [q₁ | q₂ | ··· | qₙ]       │
  │  7. Result: A = QDQᵀ                     │
  └──────────────────────────────────────────┘
```

**Key point:** Step 4 is only needed when an eigenvalue has geometric multiplicity > 1. Eigenvectors from **different** eigenvalues are automatically orthogonal (by the symmetric matrix theorem).

---

## 4. Worked Example 1: $2 \times 2$

Orthogonally diagonalise $A = \begin{pmatrix} 5 & -2 \\ -2 & 2 \end{pmatrix}$.

**Step 2:** $p(\lambda) = \lambda^2 - 7\lambda + 6 = (\lambda - 1)(\lambda - 6)$

$\lambda_1 = 1, \lambda_2 = 6$

**Step 3:**

$\lambda = 1$: $(A - I) = \begin{pmatrix} 4 & -2 \\ -2 & 1 \end{pmatrix} \xrightarrow{\text{RREF}} \begin{pmatrix} 1 & -1/2 \\ 0 & 0 \end{pmatrix}$, $\mathbf{v}_1 = (1, 2)^T$

$\lambda = 6$: $(A - 6I) = \begin{pmatrix} -1 & -2 \\ -2 & -4 \end{pmatrix} \xrightarrow{\text{RREF}} \begin{pmatrix} 1 & 2 \\ 0 & 0 \end{pmatrix}$, $\mathbf{v}_2 = (-2, 1)^T$

**Verify orthogonality:** $\mathbf{v}_1 \cdot \mathbf{v}_2 = -2 + 2 = 0$ ✓ (automatic for symmetric!)

**Step 5:** Normalise:

$$\mathbf{q}_1 = \frac{1}{\sqrt{5}}\begin{pmatrix} 1 \\ 2 \end{pmatrix}, \quad \mathbf{q}_2 = \frac{1}{\sqrt{5}}\begin{pmatrix} -2 \\ 1 \end{pmatrix}$$

**Result:**

$$A = QDQ^T = \frac{1}{\sqrt{5}}\begin{pmatrix} 1 & -2 \\ 2 & 1 \end{pmatrix}\begin{pmatrix} 1 & 0 \\ 0 & 6 \end{pmatrix}\frac{1}{\sqrt{5}}\begin{pmatrix} 1 & 2 \\ -2 & 1 \end{pmatrix}$$

**Spectral form:** $A = 1 \cdot \mathbf{q}_1\mathbf{q}_1^T + 6 \cdot \mathbf{q}_2\mathbf{q}_2^T$

---

## 5. Worked Example 2: $3 \times 3$ with Repeated Eigenvalue

Orthogonally diagonalise $A = \begin{pmatrix} 2 & 0 & 0 \\ 0 & 3 & 1 \\ 0 & 1 & 3 \end{pmatrix}$.

**Step 2:** $p(\lambda) = (2-\lambda)[(3-\lambda)^2 - 1] = (2-\lambda)(\lambda^2 - 6\lambda + 8) = (2-\lambda)(4-\lambda)(2-\lambda)$

$= (2-\lambda)^2(4-\lambda)$

$\lambda_1 = 2$ (am = 2), $\lambda_2 = 4$ (am = 1)

**Step 3:**

$\lambda = 2$: $A - 2I = \begin{pmatrix} 0 & 0 & 0 \\ 0 & 1 & 1 \\ 0 & 1 & 1 \end{pmatrix}$

Null space: $v_1$ free, $v_2 = -v_3$

$$E_2 = \text{span}\left\{\begin{pmatrix} 1 \\ 0 \\ 0 \end{pmatrix}, \begin{pmatrix} 0 \\ -1 \\ 1 \end{pmatrix}\right\}$$

These are already orthogonal! (Lucky — but in general, apply Gram-Schmidt within the eigenspace.)

$\lambda = 4$: $A - 4I = \begin{pmatrix} -2 & 0 & 0 \\ 0 & -1 & 1 \\ 0 & 1 & -1 \end{pmatrix}$, $\mathbf{v}_3 = (0, 1, 1)^T$

**Verify:** $\mathbf{v}_3 \perp E_2$? $(0,1,1) \cdot (1,0,0) = 0$ ✓, $(0,1,1) \cdot (0,-1,1) = 0$ ✓

**Step 5:** Normalise all:

$$Q = \begin{pmatrix} 1 & 0 & 0 \\ 0 & -1/\sqrt{2} & 1/\sqrt{2} \\ 0 & 1/\sqrt{2} & 1/\sqrt{2} \end{pmatrix}, \quad D = \begin{pmatrix} 2 & 0 & 0 \\ 0 & 2 & 0 \\ 0 & 0 & 4 \end{pmatrix}$$

---

## 6. Why the Spectral Theorem Works

The proof relies on two facts:
1. **Real eigenvalues** → can find at least one eigenvector $\mathbf{v}_1$
2. **$A$ maps $\mathbf{v}_1^\perp$ to itself** → can apply induction on the orthogonal complement

This second fact is the key insight: if $A\mathbf{v}_1 = \lambda_1 \mathbf{v}_1$ and $\mathbf{w} \perp \mathbf{v}_1$, then $A\mathbf{w} \perp \mathbf{v}_1$ (because $\langle A\mathbf{w}, \mathbf{v}_1 \rangle = \langle \mathbf{w}, A\mathbf{v}_1 \rangle = \lambda_1 \langle \mathbf{w}, \mathbf{v}_1 \rangle = 0$).

---

## 7. Applications of the Spectral Theorem

| Application | How spectral decomposition helps |
|---|---|
| **Principal Component Analysis (PCA)** | Eigenvalues = variances along principal axes |
| **Quadratic forms** | Classify as positive/negative definite |
| **Elastic deformation** | Principal stresses along eigenvectors |
| **Moment of inertia** | Principal axes of rotation |
| **Graph theory** | Spectral clustering via Laplacian eigenvalues |
| **Quantum mechanics** | Observables are Hermitian operators |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Spectral theorem | $A = A^T \implies A = QDQ^T$ with $Q$ orthogonal |
| Spectral decomposition | $A = \sum \lambda_i \mathbf{q}_i \mathbf{q}_i^T$ |
| Projections $P_i$ | $P_i = \mathbf{q}_i\mathbf{q}_i^T$: rank-1, orthogonal, sum to $I$ |
| Procedure | Eigenvalues → Eigenspaces → Gram-Schmidt within → Normalise |
| Key insight | $A$ maps $\mathbf{v}^\perp$ to itself for symmetric $A$ |
| Advantage | $Q^{-1} = Q^T$ (no inversion needed) |

---

## Quick Revision Questions

1. State the Spectral Theorem for real symmetric matrices.

2. Write the spectral decomposition of $A = \begin{pmatrix} 3 & 1 \\ 1 & 3 \end{pmatrix}$.

3. Why do we only need Gram-Schmidt within eigenspaces (not across them)?

4. Show that each $P_i = \mathbf{q}_i \mathbf{q}_i^T$ is idempotent and satisfies $P_iP_j = 0$ for $i \neq j$.

5. Why is the Spectral Theorem limited to symmetric (or Hermitian) matrices?

---

[← Previous: Symmetric Matrices](01-symmetric-matrices.md) · [Back to README](../README.md) · [Next: Orthogonal Matrices →](03-orthogonal-matrices.md)
