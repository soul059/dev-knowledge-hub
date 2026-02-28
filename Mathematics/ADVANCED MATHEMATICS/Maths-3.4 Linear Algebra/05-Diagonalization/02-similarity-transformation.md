# 5.2 Similarity Transformation

[← Previous: Diagonalizable Matrices](01-diagonalizable-matrices.md) · [Back to README](../README.md) · [Next: Diagonalization Procedure →](03-diagonalization-procedure.md)

---

## Chapter Overview

Diagonalisation is a special case of **similarity transformations**. This chapter explores the general theory of similarity — what properties are preserved, and how similar matrices represent the same linear transformation in different bases.

---

## 1. Definition of Similarity

> **Definition.** Matrices $A$ and $B$ are **similar** (written $A \sim B$) if there exists an invertible matrix $P$ such that:
>
> $$B = P^{-1}AP$$

Similarity is an **equivalence relation**:
- **Reflexive:** $A \sim A$ (take $P = I$)
- **Symmetric:** $A \sim B \implies B \sim A$ (use $P^{-1}$)
- **Transitive:** $A \sim B, B \sim C \implies A \sim C$ (use composition)

---

## 2. Invariants Under Similarity

> If $A \sim B$ ($B = P^{-1}AP$), the following are **identical**:

| Invariant | Why |
|-----------|-----|
| Characteristic polynomial | $\det(B - \lambda I) = \det(P^{-1}(A - \lambda I)P) = \det(A - \lambda I)$ |
| Eigenvalues | Roots of same char. poly. |
| Algebraic multiplicities | Same char. poly. → same root multiplicities |
| Geometric multiplicities | $\ker(B - \lambda I) \cong \ker(A - \lambda I)$ |
| Trace | $\text{tr}(P^{-1}AP) = \text{tr}(A)$ |
| Determinant | $\det(P^{-1}AP) = \det(A)$ |
| Rank | $\text{rank}(B) = \text{rank}(A)$ |
| Nullity | $\text{nullity}(B) = \text{nullity}(A)$ |
| Minimal polynomial | Same |

**NOT preserved:** Eigenvectors, row echelon form, individual entries, symmetry (in general).

```
  Similar matrices share:
  
  ┌──────────────────────────────────┐
  │  INVARIANTS (same for A ~ B)     │
  │                                  │
  │  • Characteristic polynomial     │
  │  • Eigenvalues + multiplicities  │
  │  • Trace, Determinant, Rank      │
  │  • Minimal polynomial            │
  └──────────────────────────────────┘
  
  ┌──────────────────────────────────┐
  │  NOT INVARIANT                   │
  │                                  │
  │  • Eigenvectors (change with P)  │
  │  • Individual matrix entries     │
  │  • Symmetry (in general)         │
  └──────────────────────────────────┘
```

---

## 3. The Change-of-Basis Interpretation

Similar matrices represent the **same linear transformation** in different bases.

If $T: V \to V$ is a linear transformation and $\mathcal{B}, \mathcal{B}'$ are two bases:

$$[T]_{\mathcal{B}'} = P^{-1} [T]_{\mathcal{B}} P$$

where $P$ is the change-of-basis matrix from $\mathcal{B}'$ to $\mathcal{B}$.

```
  Same transformation, different viewpoints:
  
  Basis B:    [T]_B = A
                 │
                 │  P⁻¹(·)P
                 ▼
  Basis B':   [T]_B' = P⁻¹AP = B
  
  A and B are similar — they "see" the same T
```

**Diagonalisation** is the special case where $\mathcal{B}'$ is the eigenvector basis:

$$[T]_{\text{eigenbasis}} = D = \text{diag}(\lambda_1, \dots, \lambda_n)$$

---

## 4. Checking Similarity

Two matrices are similar if and only if they have:
1. The same characteristic polynomial, **AND**
2. The same geometric multiplicities for each eigenvalue, **AND**
3. The same **Jordan normal form** (the definitive test)

**Warning:** Same eigenvalues and trace/det are **necessary** but not **sufficient**.

**Example:** $A = \begin{pmatrix} 2 & 1 \\ 0 & 2 \end{pmatrix}$ and $B = \begin{pmatrix} 2 & 0 \\ 0 & 2 \end{pmatrix}$

Both have $p(\lambda) = (2-\lambda)^2$, same trace (4), same det (4).

But $\text{gm}_A(2) = 1$ while $\text{gm}_B(2) = 2$. Therefore $A \not\sim B$.

---

## 5. Properties of Similar Matrices

> If $A \sim B$, then:
> 1. $A^k \sim B^k$ for all $k \geq 0$
> 2. $p(A) \sim p(B)$ for any polynomial $p$
> 3. If $A$ is invertible, $A^{-1} \sim B^{-1}$
> 4. $A + cI \sim B + cI$

---

## 6. Trace as a Similarity Invariant: Proof

$$\text{tr}(P^{-1}AP) = \text{tr}((P^{-1}A)P) = \text{tr}(P(P^{-1}A)) = \text{tr}(A)$$

using the cyclic property: $\text{tr}(XY) = \text{tr}(YX)$.

---

## 7. Worked Example: Finding $P$

Given $A = \begin{pmatrix} 1 & 3 \\ 2 & 2 \end{pmatrix}$, diagonalise and find similarity transformation.

$p(\lambda) = \lambda^2 - 3\lambda - 4 = (\lambda - 4)(\lambda + 1)$

$\lambda_1 = 4$: $\mathbf{v}_1 = (1, 1)^T$ $\qquad$ $\lambda_2 = -1$: $\mathbf{v}_2 = (3, -2)^T$

$$P = \begin{pmatrix} 1 & 3 \\ 1 & -2 \end{pmatrix}, \quad P^{-1} = \frac{1}{-5}\begin{pmatrix} -2 & -3 \\ -1 & 1 \end{pmatrix} = \begin{pmatrix} 2/5 & 3/5 \\ 1/5 & -1/5 \end{pmatrix}$$

$$P^{-1}AP = \begin{pmatrix} 4 & 0 \\ 0 & -1 \end{pmatrix} = D$$

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Similarity | $B = P^{-1}AP$ for some invertible $P$ |
| Equivalence relation | Reflexive, symmetric, transitive |
| Invariants | Eigenvalues, trace, det, rank, char. poly., min. poly. |
| Not preserved | Eigenvectors, entries, symmetry |
| Interpretation | Same linear transformation, different bases |
| Diagonalisation | $P^{-1}AP = D$ where $P$ = eigenvector matrix |
| Definitive test | Same Jordan normal form |

---

## Quick Revision Questions

1. Show that similarity is an equivalence relation.

2. If $A \sim B$, prove $\det(A) = \det(B)$.

3. Give two $2 \times 2$ matrices with the same eigenvalues that are NOT similar.

4. If $A \sim B$ and $A$ is invertible, show $A^{-1} \sim B^{-1}$.

5. Interpret similarity in terms of change of basis for a linear transformation.

---

[← Previous: Diagonalizable Matrices](01-diagonalizable-matrices.md) · [Back to README](../README.md) · [Next: Diagonalization Procedure →](03-diagonalization-procedure.md)
