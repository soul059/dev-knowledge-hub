# 4.2 The Characteristic Polynomial

[← Previous: Definitions](01-definitions.md) · [Back to README](../README.md) · [Next: Finding Eigenvalues & Eigenvectors →](03-finding-eigenvalues-eigenvectors.md)

---

## Chapter Overview

The **characteristic polynomial** is the key algebraic tool for finding eigenvalues. This chapter develops the polynomial, its properties, and connections to trace and determinant.

---

## 1. Definition

> **Definition.** The **characteristic polynomial** of an $n \times n$ matrix $A$ is:
>
> $$p(\lambda) = \det(A - \lambda I)$$
>
> The equation $\det(A - \lambda I) = 0$ is the **characteristic equation**.

$p(\lambda)$ is a polynomial of degree $n$ in $\lambda$.

---

## 2. Computing the Characteristic Polynomial

### For $2 \times 2$ Matrices

$$A = \begin{pmatrix} a & b \\ c & d \end{pmatrix}$$

$$p(\lambda) = \det \begin{pmatrix} a-\lambda & b \\ c & d-\lambda \end{pmatrix} = \lambda^2 - (a+d)\lambda + (ad-bc)$$

$$\boxed{p(\lambda) = \lambda^2 - \text{tr}(A)\lambda + \det(A)}$$

### For $3 \times 3$ Matrices

$$A = \begin{pmatrix} a_{11} & a_{12} & a_{13} \\ a_{21} & a_{22} & a_{23} \\ a_{31} & a_{32} & a_{33} \end{pmatrix}$$

$$p(\lambda) = -\lambda^3 + \text{tr}(A)\lambda^2 - (\text{sum of } 2\times 2 \text{ cofactors})\lambda + \det(A)$$

**Worked Example:**

$$A = \begin{pmatrix} 2 & 1 & 0 \\ 0 & 3 & 1 \\ 0 & 0 & 5 \end{pmatrix}$$

Since $A$ is upper triangular:

$$p(\lambda) = (2 - \lambda)(3 - \lambda)(5 - \lambda)$$

Eigenvalues: $\lambda = 2, 3, 5$ (the diagonal entries).

---

## 3. General Structure

For $n \times n$ matrix $A$, the characteristic polynomial has the form:

$$p(\lambda) = (-1)^n \lambda^n + (-1)^{n-1} \text{tr}(A) \lambda^{n-1} + \dots + \det(A)$$

The coefficients are related to the **principal minors** of $A$.

Special values:
- $p(0) = \det(A)$
- Leading coefficient: $(-1)^n$
- Coefficient of $\lambda^{n-1}$: $(-1)^{n-1} \text{tr}(A)$

---

## 4. Trace, Determinant, and Eigenvalues

If $\lambda_1, \lambda_2, \dots, \lambda_n$ are the eigenvalues (counted with multiplicity), then:

$$p(\lambda) = (-1)^n (\lambda - \lambda_1)(\lambda - \lambda_2) \cdots (\lambda - \lambda_n)$$

Expanding and comparing coefficients:

| Relationship | Formula |
|:---:|:---:|
| Sum of eigenvalues | $\lambda_1 + \lambda_2 + \cdots + \lambda_n = \text{tr}(A)$ |
| Product of eigenvalues | $\lambda_1 \cdot \lambda_2 \cdots \lambda_n = \det(A)$ |

```
  Characteristic polynomial encodes everything:
  
  p(λ) = (-1)ⁿ(λ - λ₁)(λ - λ₂)···(λ - λₙ)
              │         │              │
              ▼         ▼              ▼
           ┌──────────────────────────────────┐
           │  Roots = Eigenvalues              │
           │  Sum of roots = tr(A)             │
           │  Product of roots = det(A)        │
           │  p(0) = det(A)                    │
           └──────────────────────────────────┘
```

---

## 5. Algebraic Multiplicity

> **Definition.** The **algebraic multiplicity** of eigenvalue $\lambda_0$ is the multiplicity of $\lambda_0$ as a root of the characteristic polynomial.

**Example:**

$$p(\lambda) = (\lambda - 2)^3 (\lambda - 5)$$

- $\lambda = 2$ has algebraic multiplicity $\text{am}(2) = 3$
- $\lambda = 5$ has algebraic multiplicity $\text{am}(5) = 1$

Note: $\sum \text{am}(\lambda_i) = n$ (degree of $p(\lambda)$), counting over all distinct eigenvalues.

---

## 6. Cayley-Hamilton Theorem

> **Theorem (Cayley-Hamilton).** Every square matrix satisfies its own characteristic equation:
>
> $$p(A) = 0$$

If $p(\lambda) = \lambda^2 - 5\lambda + 6$, then $A^2 - 5A + 6I = 0$.

### Applications of Cayley-Hamilton

1. **Computing $A^{-1}$:** From $A^2 - 5A + 6I = 0$: $A(A - 5I) = -6I$, so $A^{-1} = \frac{1}{6}(5I - A)$.

2. **Reducing powers of $A$:** Express $A^k$ in terms of $I, A, \dots, A^{n-1}$.

**Worked Example:**

$$A = \begin{pmatrix} 1 & 2 \\ 3 & 4 \end{pmatrix}$$

$p(\lambda) = \lambda^2 - 5\lambda - 2$

By Cayley-Hamilton: $A^2 = 5A + 2I$

$$A^2 = 5\begin{pmatrix} 1 & 2 \\ 3 & 4 \end{pmatrix} + 2\begin{pmatrix} 1 & 0 \\ 0 & 1 \end{pmatrix} = \begin{pmatrix} 7 & 10 \\ 15 & 22 \end{pmatrix}$$

Verify: $A^2 = \begin{pmatrix} 1 & 2 \\ 3 & 4 \end{pmatrix}\begin{pmatrix} 1 & 2 \\ 3 & 4 \end{pmatrix} = \begin{pmatrix} 7 & 10 \\ 15 & 22 \end{pmatrix}$ ✓

---

## 7. Similar Matrices Have the Same Characteristic Polynomial

> **Theorem.** If $B = P^{-1}AP$, then $p_B(\lambda) = p_A(\lambda)$.

**Proof:**

$$\det(B - \lambda I) = \det(P^{-1}AP - \lambda P^{-1}P) = \det(P^{-1}(A - \lambda I)P)$$
$$= \det(P^{-1}) \det(A - \lambda I) \det(P) = \det(A - \lambda I) \quad \blacksquare$$

**Corollary:** Similar matrices have the same eigenvalues, trace, and determinant.

---

## 8. Characteristic Polynomial of Special Matrices

| Matrix Type | Characteristic Polynomial |
|---|---|
| $\text{diag}(d_1, \dots, d_n)$ | $(d_1 - \lambda)(d_2 - \lambda)\cdots(d_n - \lambda)$ |
| Upper/Lower triangular | Same as diagonal |
| $2 \times 2$ general | $\lambda^2 - \text{tr}(A)\lambda + \det(A)$ |
| Companion matrix | Directly reads off any desired polynomial |
| $A$ and $A^T$ | Identical characteristic polynomials |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Characteristic polynomial | $p(\lambda) = \det(A - \lambda I)$, degree $n$ |
| Characteristic equation | $\det(A - \lambda I) = 0$ |
| $2 \times 2$ shortcut | $p(\lambda) = \lambda^2 - \text{tr}(A)\lambda + \det(A)$ |
| Algebraic multiplicity | Multiplicity of $\lambda$ as root of $p$ |
| Trace relation | $\text{tr}(A) = \sum \lambda_i$ |
| Determinant relation | $\det(A) = \prod \lambda_i$ |
| Cayley-Hamilton | $p(A) = 0$ (matrix satisfies its own char. poly.) |
| Similar matrices | Same characteristic polynomial |

---

## Quick Revision Questions

1. Find the characteristic polynomial of $A = \begin{pmatrix} 3 & -1 \\ 1 & 1 \end{pmatrix}$.

2. A $3 \times 3$ matrix has eigenvalues $1, 2, 3$. Find $\text{tr}(A)$ and $\det(A)$.

3. State the Cayley-Hamilton theorem. Use it to find $A^{-1}$ for the matrix in Q1.

4. Prove that $A$ and $A^T$ have the same characteristic polynomial.

5. A matrix has $p(\lambda) = (\lambda - 1)^2(\lambda + 3)$. What are the eigenvalues and their algebraic multiplicities?

---

[← Previous: Definitions](01-definitions.md) · [Back to README](../README.md) · [Next: Finding Eigenvalues & Eigenvectors →](03-finding-eigenvalues-eigenvectors.md)
