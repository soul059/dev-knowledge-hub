# 5.5 Non-Diagonalizable Matrices and Jordan Form

[← Previous: Applications — Matrix Powers](04-applications-matrix-powers.md) · [Back to README](../README.md) · [Next: Unit 6 — Symmetric Matrices →](../06-Orthogonal-Diagonalization/01-symmetric-matrices.md)

---

## Chapter Overview

Not every matrix is diagonalisable. When $\text{gm} < \text{am}$ for some eigenvalue, we turn to the **Jordan normal form** — the "next best thing" after a diagonal matrix. This chapter introduces Jordan blocks, generalised eigenvectors, and the structure theorem.

---

## 1. When Diagonalisation Fails

A matrix fails to be diagonalisable precisely when there are not enough independent eigenvectors: some eigenvalue $\lambda$ has $\text{gm}(\lambda) < \text{am}(\lambda)$.

**Example:** $A = \begin{pmatrix} 5 & 1 \\ 0 & 5 \end{pmatrix}$

$p(\lambda) = (5-\lambda)^2$, $\text{am}(5) = 2$

$A - 5I = \begin{pmatrix} 0 & 1 \\ 0 & 0 \end{pmatrix}$, rank = 1, $\text{gm}(5) = 1$

Only one eigenvector! We need a **generalised eigenvector** to fill the gap.

---

## 2. Generalised Eigenvectors

> **Definition.** A **generalised eigenvector** of rank $k$ for eigenvalue $\lambda$ is a nonzero vector $\mathbf{w}$ satisfying:
>
> $$(A - \lambda I)^k \mathbf{w} = \mathbf{0} \quad \text{but} \quad (A - \lambda I)^{k-1} \mathbf{w} \neq \mathbf{0}$$

Ordinary eigenvectors have rank 1: $(A - \lambda I)\mathbf{v} = \mathbf{0}$.

A rank 2 generalised eigenvector satisfies: $(A - \lambda I)^2\mathbf{w} = \mathbf{0}$ but $(A - \lambda I)\mathbf{w} \neq \mathbf{0}$.

```
  Chain of generalised eigenvectors:
  
  w₂ ──(A−λI)──> w₁ ──(A−λI)──> 0
  
  where w₁ = (A−λI)w₂ is an eigenvector
  and w₂ is a generalised eigenvector of rank 2
```

---

## 3. Jordan Blocks

> **Definition.** A **Jordan block** of size $m$ for eigenvalue $\lambda$ is:
>
> $$J_m(\lambda) = \begin{pmatrix} \lambda & 1 & 0 & \cdots & 0 \\ 0 & \lambda & 1 & \cdots & 0 \\ 0 & 0 & \lambda & \cdots & 0 \\ \vdots & & & \ddots & 1 \\ 0 & 0 & 0 & \cdots & \lambda \end{pmatrix}_{m \times m}$$

Examples:

$$J_1(3) = (3), \quad J_2(3) = \begin{pmatrix} 3 & 1 \\ 0 & 3 \end{pmatrix}, \quad J_3(3) = \begin{pmatrix} 3 & 1 & 0 \\ 0 & 3 & 1 \\ 0 & 0 & 3 \end{pmatrix}$$

A $1 \times 1$ Jordan block is just a diagonal entry — so diagonalisable matrices are the special case where all Jordan blocks are $1 \times 1$.

---

## 4. Jordan Normal Form

> **Theorem (Jordan Normal Form).** Every square matrix $A$ over $\mathbb{C}$ is similar to a **Jordan matrix**:
>
> $$J = P^{-1}AP = \begin{pmatrix} J_{m_1}(\lambda_1) & & \\ & J_{m_2}(\lambda_2) & \\ & & \ddots & \\ & & & J_{m_r}(\lambda_r) \end{pmatrix}$$

The Jordan form is unique up to permutation of blocks.

```
  Jordan Normal Form structure:
  
  ┌─────────────────────────────────┐
  │ J_m₁(λ₁)  0       0      0     │
  │   0     J_m₂(λ₂)  0      0     │
  │   0       0     J_m₃(λ₃) 0     │
  │   0       0       0    J_m₄(λ₄)│
  └─────────────────────────────────┘
  
  Each block is either 1×1 (diagonal entry)
  or larger (with 1's on superdiagonal)
```

---

## 5. Relationship Between Jordan Form and Multiplicities

For each eigenvalue $\lambda$:
- **am($\lambda$)** = total size of all Jordan blocks for $\lambda$
- **gm($\lambda$)** = number of Jordan blocks for $\lambda$
- Diagonalisable iff all blocks are $1 \times 1$ (gm = am)

| Jordan structure for $\lambda$ | am | gm |
|---|:---:|:---:|
| $J_1(\lambda), J_1(\lambda), J_1(\lambda)$ | 3 | 3 (diagonalisable) |
| $J_2(\lambda), J_1(\lambda)$ | 3 | 2 |
| $J_3(\lambda)$ | 3 | 1 |

---

## 6. Worked Example: Finding Jordan Form

$$A = \begin{pmatrix} 5 & 4 & 2 & 1 \\ 0 & 1 & -1 & -1 \\ -1 & -1 & 3 & 0 \\ 1 & 1 & -1 & 2 \end{pmatrix}$$

Suppose $p(\lambda) = (\lambda - 1)^2(\lambda - 4)^2$.

For $\lambda = 1$: $\text{am} = 2$. Compute rank of $A - I$ → if $\text{gm} = 1$: Jordan block $J_2(1)$; if $\text{gm} = 2$: two $J_1(1)$ blocks.

For $\lambda = 4$: Same analysis.

---

## 7. Powers of Jordan Blocks

$$J_m(\lambda)^k = \sum_{j=0}^{m-1} \binom{k}{j} \lambda^{k-j} N^j$$

where $N$ is the nilpotent part (1's on superdiagonal).

For a $2 \times 2$ block:

$$J_2(\lambda)^k = \begin{pmatrix} \lambda^k & k\lambda^{k-1} \\ 0 & \lambda^k \end{pmatrix}$$

For a $3 \times 3$ block:

$$J_3(\lambda)^k = \begin{pmatrix} \lambda^k & k\lambda^{k-1} & \binom{k}{2}\lambda^{k-2} \\ 0 & \lambda^k & k\lambda^{k-1} \\ 0 & 0 & \lambda^k \end{pmatrix}$$

---

## 8. Comparison: Diagonal vs Jordan

| Property | Diagonal | Jordan |
|----------|----------|--------|
| Exists for | Diagonalisable matrices | **All** matrices (over $\mathbb{C}$) |
| Off-diagonal | All zeros | Some 1's on superdiagonal |
| $A^k$ computation | $\lambda_i^k$ | Involves binomial coefficients |
| Eigenvectors | $n$ independent | Need generalised eigenvectors |
| Uniqueness | Up to column order | Up to block order |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Non-diagonalisable | $\text{gm} < \text{am}$ for some eigenvalue |
| Generalised eigenvector | $(A - \lambda I)^k \mathbf{w} = 0$, $(A - \lambda I)^{k-1}\mathbf{w} \neq 0$ |
| Jordan block $J_m(\lambda)$ | $m \times m$ with $\lambda$ on diagonal, 1's above |
| Jordan normal form | Block diagonal with Jordan blocks |
| Existence | Every matrix has a JNF (over $\mathbb{C}$) |
| Uniqueness | Unique up to block ordering |
| gm = number of blocks | For each eigenvalue |
| am = total block size | For each eigenvalue |

---

## Quick Revision Questions

1. Find the Jordan form of $A = \begin{pmatrix} 3 & 1 \\ 0 & 3 \end{pmatrix}$.

2. What are all possible Jordan forms for a $3 \times 3$ matrix with eigenvalue $\lambda = 2$ of algebraic multiplicity 3?

3. Compute $J_2(5)^{10}$.

4. A $4 \times 4$ matrix has $p(\lambda) = (\lambda-1)^4$ and $\text{gm}(1) = 2$. What are the possible Jordan forms?

5. Explain why diagonalisable matrices are the special case of Jordan form where all blocks are $1 \times 1$.

---

[← Previous: Applications — Matrix Powers](04-applications-matrix-powers.md) · [Back to README](../README.md) · [Next: Unit 6 — Symmetric Matrices →](../06-Orthogonal-Diagonalization/01-symmetric-matrices.md)
