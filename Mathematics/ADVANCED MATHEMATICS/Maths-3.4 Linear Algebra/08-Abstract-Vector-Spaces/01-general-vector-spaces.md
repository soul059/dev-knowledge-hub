# 8.1 General Vector Spaces

[← Previous: Unit 7 — Data Compression](../07-SVD-and-Applications/04-data-compression.md) · [Back to README](../README.md) · [Next: Function Spaces →](02-function-spaces.md)

---

## Chapter Overview

Until now, we have mainly worked in $\mathbb{R}^n$. But the axioms of a vector space apply to far more than just columns of numbers — polynomials, matrices, functions, sequences, and even more abstract objects form vector spaces. This chapter explores the full generality.

---

## 1. The Axioms Revisited

A **vector space** over $\mathbb{R}$ is a set $V$ with addition and scalar multiplication satisfying **10 axioms**:

| # | Axiom | Statement |
|:---:|---|---|
| A1 | Closure (addition) | $\mathbf{u} + \mathbf{v} \in V$ |
| A2 | Commutativity | $\mathbf{u} + \mathbf{v} = \mathbf{v} + \mathbf{u}$ |
| A3 | Associativity | $(\mathbf{u} + \mathbf{v}) + \mathbf{w} = \mathbf{u} + (\mathbf{v} + \mathbf{w})$ |
| A4 | Zero vector | $\exists \mathbf{0}: \mathbf{v} + \mathbf{0} = \mathbf{v}$ |
| A5 | Additive inverse | $\exists (-\mathbf{v}): \mathbf{v} + (-\mathbf{v}) = \mathbf{0}$ |
| S1 | Closure (scalar mult.) | $c\mathbf{v} \in V$ |
| S2 | Distributivity (scalar) | $c(\mathbf{u} + \mathbf{v}) = c\mathbf{u} + c\mathbf{v}$ |
| S3 | Distributivity (vector) | $(c + d)\mathbf{v} = c\mathbf{v} + d\mathbf{v}$ |
| S4 | Associativity (scalar) | $c(d\mathbf{v}) = (cd)\mathbf{v}$ |
| S5 | Identity | $1\mathbf{v} = \mathbf{v}$ |

The power of this abstraction: **any** set satisfying these axioms inherits all theorems about vector spaces — basis, dimension, linear transformations, etc.

---

## 2. Catalogue of Vector Spaces

```
  The vector space universe:
  
  ┌───────────────────────────────────────────────────┐
  │                VECTOR SPACES                       │
  ├───────────────────────────────────────────────────┤
  │                                                   │
  │  Finite-dimensional:                              │
  │  • ℝⁿ (column vectors)                           │
  │  • M_{m×n}(ℝ) (matrices)       dim = mn          │
  │  • P_n(ℝ) (polynomials ≤ n)    dim = n+1         │
  │  • ℂ (as real vector space)     dim = 2           │
  │                                                   │
  │  Infinite-dimensional:                            │
  │  • P(ℝ) (all polynomials)      dim = ∞           │
  │  • C[a,b] (continuous funcs)   dim = ∞           │
  │  • C^∞(ℝ) (smooth functions)   dim = ∞           │
  │  • ℓ² (square-summable seqs)   dim = ∞           │
  │  • Solutions to y'' + y = 0    dim = 2            │
  │                                                   │
  └───────────────────────────────────────────────────┘
```

---

## 3. Matrix Spaces $M_{m \times n}(\mathbb{R})$

The set of all $m \times n$ real matrices with standard addition and scalar multiplication.

**Dimension:** $mn$

**Standard basis for $M_{2 \times 2}$:**

$$E_{11} = \begin{pmatrix} 1 & 0 \\ 0 & 0 \end{pmatrix}, E_{12} = \begin{pmatrix} 0 & 1 \\ 0 & 0 \end{pmatrix}, E_{21} = \begin{pmatrix} 0 & 0 \\ 1 & 0 \end{pmatrix}, E_{22} = \begin{pmatrix} 0 & 0 \\ 0 & 1 \end{pmatrix}$$

**Subspaces of $M_{n \times n}$:**

| Subspace | Definition | Dimension |
|----------|-----------|:---------:|
| Symmetric matrices | $A = A^T$ | $\frac{n(n+1)}{2}$ |
| Skew-symmetric matrices | $A = -A^T$ | $\frac{n(n-1)}{2}$ |
| Diagonal matrices | $a_{ij} = 0$ for $i \neq j$ | $n$ |
| Upper triangular | $a_{ij} = 0$ for $i > j$ | $\frac{n(n+1)}{2}$ |
| Trace-zero matrices | $\text{tr}(A) = 0$ | $n^2 - 1$ |

**Key decomposition:** Every square matrix decomposes uniquely as:

$$A = \underbrace{\frac{A + A^T}{2}}_{\text{symmetric}} + \underbrace{\frac{A - A^T}{2}}_{\text{skew-symmetric}}$$

$$M_{n \times n} = \text{Sym}_n \oplus \text{Skew}_n$$

---

## 4. Polynomial Spaces $\mathcal{P}_n(\mathbb{R})$

$\mathcal{P}_n$ = polynomials of degree $\leq n$ with real coefficients.

$$p(x) = a_0 + a_1 x + a_2 x^2 + \dots + a_n x^n$$

**Dimension:** $n + 1$

**Standard basis:** $\{1, x, x^2, \dots, x^n\}$

**Zero vector:** the zero polynomial ($p(x) = 0$ for all $x$)

**Subspaces:**
- Even polynomials: $\{a_0 + a_2 x^2 + a_4 x^4 + \dots\}$
- Odd polynomials: $\{a_1 x + a_3 x^3 + \dots\}$

---

## 5. Coordinate Vectors

> **Definition.** If $\mathcal{B} = \{\mathbf{b}_1, \dots, \mathbf{b}_n\}$ is an ordered basis for $V$ and $\mathbf{v} = c_1\mathbf{b}_1 + \dots + c_n\mathbf{b}_n$, then the **coordinate vector** of $\mathbf{v}$ relative to $\mathcal{B}$ is:
>
> $$[\mathbf{v}]_{\mathcal{B}} = \begin{pmatrix} c_1 \\ c_2 \\ \vdots \\ c_n \end{pmatrix} \in \mathbb{R}^n$$

This creates an **isomorphism** $V \cong \mathbb{R}^n$ — every $n$-dimensional vector space "is" $\mathbb{R}^n$ in disguise!

**Example:** In $\mathcal{P}_2$ with basis $\{1, x, x^2\}$:

$$p(x) = 3 - 2x + 5x^2 \quad \leftrightarrow \quad [p]_{\mathcal{B}} = \begin{pmatrix} 3 \\ -2 \\ 5 \end{pmatrix}$$

---

## 6. Linear Transformations on Abstract Spaces

**Example:** Differentiation $D: \mathcal{P}_3 \to \mathcal{P}_2$

$$D(a_0 + a_1 x + a_2 x^2 + a_3 x^3) = a_1 + 2a_2 x + 3a_3 x^2$$

Matrix representation (standard bases):

$$[D] = \begin{pmatrix} 0 & 1 & 0 & 0 \\ 0 & 0 & 2 & 0 \\ 0 & 0 & 0 & 3 \end{pmatrix}$$

**Kernel:** $\ker(D) = \{a_0 : a_0 \in \mathbb{R}\}$ (constant polynomials), $\dim = 1$

**Range:** $\text{Im}(D) = \mathcal{P}_2$, $\dim = 3$

Rank-Nullity: $1 + 3 = 4 = \dim(\mathcal{P}_3)$ ✓

---

## 7. Direct Sums of Abstract Spaces

> **Theorem.** $V = W_1 \oplus W_2$ iff every $\mathbf{v} \in V$ can be written **uniquely** as $\mathbf{v} = \mathbf{w}_1 + \mathbf{w}_2$ with $\mathbf{w}_i \in W_i$ and $W_1 \cap W_2 = \{\mathbf{0}\}$.

**Example in $M_{2 \times 2}$:**

$$\begin{pmatrix} a & b \\ c & d \end{pmatrix} = \underbrace{\begin{pmatrix} a & \frac{b+c}{2} \\ \frac{b+c}{2} & d \end{pmatrix}}_{\text{Sym}_2} + \underbrace{\begin{pmatrix} 0 & \frac{b-c}{2} \\ \frac{c-b}{2} & 0 \end{pmatrix}}_{\text{Skew}_2}$$

$\dim(M_{2\times 2}) = 4 = 3 + 1 = \dim(\text{Sym}_2) + \dim(\text{Skew}_2)$ ✓

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Abstract vector space | Any set satisfying the 10 axioms |
| $M_{m \times n}$ | Matrices as vectors; $\dim = mn$ |
| $\mathcal{P}_n$ | Polynomials of degree $\leq n$; $\dim = n + 1$ |
| Coordinate vector | $[\mathbf{v}]_{\mathcal{B}} \in \mathbb{R}^n$: translates abstract → concrete |
| Isomorphism | Every $n$-dim space $\cong \mathbb{R}^n$ |
| Differentiation | Linear transformation on polynomial spaces |
| Direct sum | $V = W_1 \oplus W_2$: unique decomposition |

---

## Quick Revision Questions

1. Show that $M_{2 \times 2}(\mathbb{R})$ satisfies all 10 vector space axioms.

2. What is the dimension of the space of $3 \times 3$ symmetric matrices? Give a basis.

3. Find the coordinate vector of $p(x) = 2 + 3x - x^2$ with respect to the basis $\{1, 1+x, 1+x+x^2\}$.

4. Write the matrix of the differentiation operator $D: \mathcal{P}_2 \to \mathcal{P}_1$ with respect to standard bases.

5. Prove that $\mathcal{P}_n = \text{Even}_n \oplus \text{Odd}_n$ (even and odd polynomials).

---

[← Previous: Unit 7 — Data Compression](../07-SVD-and-Applications/04-data-compression.md) · [Back to README](../README.md) · [Next: Function Spaces →](02-function-spaces.md)
