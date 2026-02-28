# 6.4 Quadratic Forms

[← Previous: Orthogonal Matrices](03-orthogonal-matrices.md) · [Back to README](../README.md) · [Next: Unit 7 — Singular Value Decomposition →](../07-SVD-and-Applications/01-singular-value-decomposition.md)

---

## Chapter Overview

A **quadratic form** is a polynomial where every term has degree 2. Quadratic forms are intimately connected to symmetric matrices. Orthogonal diagonalisation eliminates the cross terms, revealing the geometric shape of the form.

---

## 1. Definition

> **Definition.** A **quadratic form** on $\mathbb{R}^n$ is a function $Q: \mathbb{R}^n \to \mathbb{R}$ of the form:
>
> $$Q(\mathbf{x}) = \mathbf{x}^T A \mathbf{x}$$
>
> where $A$ is a symmetric matrix (we can always assume symmetry).

### In $\mathbb{R}^2$:

$$Q(x_1, x_2) = \begin{pmatrix} x_1 & x_2 \end{pmatrix}\begin{pmatrix} a & b \\ b & c \end{pmatrix}\begin{pmatrix} x_1 \\ x_2 \end{pmatrix} = ax_1^2 + 2bx_1 x_2 + cx_2^2$$

### In $\mathbb{R}^3$:

$$Q(x_1, x_2, x_3) = a_{11}x_1^2 + a_{22}x_2^2 + a_{33}x_3^2 + 2a_{12}x_1x_2 + 2a_{13}x_1x_3 + 2a_{23}x_2x_3$$

**Rule:** Coefficients of $x_i^2$ go on the diagonal; coefficients of $x_i x_j$ are split equally between $a_{ij}$ and $a_{ji}$.

---

## 2. From Quadratic Form to Matrix

**Example:** Write $Q = 3x^2 + 4xy - y^2$ in matrix form.

Coefficient of $x^2 = 3$ → $a_{11} = 3$
Coefficient of $y^2 = -1$ → $a_{22} = -1$
Coefficient of $xy = 4$ → $a_{12} = a_{21} = 2$

$$A = \begin{pmatrix} 3 & 2 \\ 2 & -1 \end{pmatrix}, \quad Q = \mathbf{x}^T A \mathbf{x}$$

---

## 3. Classification by Eigenvalues

| Classification | Eigenvalue condition | In $\mathbb{R}^2$ |
|---|---|---|
| **Positive definite** | All $\lambda_i > 0$ | Ellipse |
| **Positive semi-definite** | All $\lambda_i \geq 0$ | Degenerate ellipse |
| **Negative definite** | All $\lambda_i < 0$ | Upside-down bowl |
| **Negative semi-definite** | All $\lambda_i \leq 0$ | Degenerate |
| **Indefinite** | Mixed signs | Hyperbola/saddle |

```
  Level curves Q(x) = 1:
  
  Positive definite:     Indefinite:         Negative definite:
  (both λ > 0)          (λ₁ > 0 > λ₂)       (both λ < 0)
  
      ╭───╮               \   /              Q = 1 has NO
     │     │                \ /               solution
     │  ·  │                 ·                (Q = -1 gives
     │     │                / \               an ellipse)
      ╰───╯               /   \
   
    Ellipse              Hyperbola
```

---

## 4. Principal Axis Theorem

> **Theorem (Principal Axis Theorem).** Every quadratic form can be transformed to one with **no cross terms** by an orthogonal change of variables.

If $A = QDQ^T$, let $\mathbf{x} = Q\mathbf{y}$ (rotation to principal axes):

$$Q(\mathbf{x}) = \mathbf{x}^T A \mathbf{x} = (Q\mathbf{y})^T A (Q\mathbf{y}) = \mathbf{y}^T (Q^T A Q) \mathbf{y} = \mathbf{y}^T D \mathbf{y}$$

$$= \lambda_1 y_1^2 + \lambda_2 y_2^2 + \dots + \lambda_n y_n^2$$

No cross terms! The eigenvalues determine the nature of the form.

```
  Original form:         After rotation x = Qy:
  
  Q(x) = 5x₁² + 4x₁x₂ + 5x₂²    Q(y) = 3y₁² + 7y₂²
  
      x₂                              y₂
       │  ╱╲                            │
       │╱    ╲  (tilted               ──┤    (aligned
      ╱│      ╲  ellipse)              │╲    with axes)
     ╱ │       ╲                       │ ╲
  ──╱──┼────────→ x₁              ─────┼──→ y₁
       │                               │
  
  Cross terms present           Pure squares only
```

---

## 5. Worked Example

Classify and find principal axes for $Q = 2x_1^2 + 4x_1x_2 + 5x_2^2$.

**Step 1:** Associated matrix:

$$A = \begin{pmatrix} 2 & 2 \\ 2 & 5 \end{pmatrix}$$

**Step 2:** Eigenvalues: $p(\lambda) = \lambda^2 - 7\lambda + 6 = (\lambda - 1)(\lambda - 6)$

$\lambda_1 = 1, \lambda_2 = 6$ → both positive → **positive definite** (ellipse)

**Step 3:** Eigenvectors (normalised):

$\lambda = 1$: $\mathbf{q}_1 = \frac{1}{\sqrt{5}}(2, -1)^T$

$\lambda = 6$: $\mathbf{q}_2 = \frac{1}{\sqrt{5}}(1, 2)^T$

**Step 4:** In principal axes ($\mathbf{x} = Q\mathbf{y}$):

$$Q = y_1^2 + 6y_2^2$$

This is an ellipse with semi-axes of lengths $1$ and $1/\sqrt{6}$.

---

## 6. Conic Sections via Quadratic Forms

Every conic section $ax^2 + bxy + cy^2 + dx + ey + f = 0$ has its type determined by the symmetric matrix:

$$A = \begin{pmatrix} a & b/2 \\ b/2 & c \end{pmatrix}$$

| $\det(A)$ | $\text{tr}(A)$ | Conic |
|:---:|:---:|---|
| $> 0$ | $> 0$ | Ellipse |
| $> 0$ | $< 0$ | Ellipse (upside-down) |
| $< 0$ | any | Hyperbola |
| $= 0$ | any | Parabola |

---

## 7. Constrained Optimisation (Rayleigh Quotient)

> **Rayleigh Quotient.** For symmetric $A$ and $\mathbf{x} \neq \mathbf{0}$:
>
> $$R(\mathbf{x}) = \frac{\mathbf{x}^T A \mathbf{x}}{\mathbf{x}^T \mathbf{x}}$$

> **Theorem.** $\lambda_{\min} \leq R(\mathbf{x}) \leq \lambda_{\max}$
>
> with equality at the corresponding eigenvectors.

This tells us:
- Maximum of $Q(\mathbf{x})$ on unit sphere = $\lambda_{\max}$, achieved at eigenvector for $\lambda_{\max}$
- Minimum of $Q(\mathbf{x})$ on unit sphere = $\lambda_{\min}$, achieved at eigenvector for $\lambda_{\min}$

---

## 8. Sylvester's Law of Inertia

> **Theorem.** The number of positive, negative, and zero eigenvalues of a symmetric matrix is invariant under congruence ($B = P^TAP$ for any invertible $P$).
>
> The triple $(\text{pos}, \text{neg}, \text{zero})$ is the **signature** of the quadratic form.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Quadratic form | $Q(\mathbf{x}) = \mathbf{x}^T A \mathbf{x}$, $A$ symmetric |
| Matrix from form | Diagonal = squared coefficients, off-diagonal = half cross-term |
| Principal Axis Theorem | $Q = \lambda_1 y_1^2 + \dots + \lambda_n y_n^2$ after rotation |
| Classification | Signs of eigenvalues determine definiteness |
| Positive definite | All $\lambda_i > 0$ → ellipsoid |
| Indefinite | Mixed signs → hyperboloid/saddle |
| Rayleigh quotient | $\lambda_{\min} \leq \frac{\mathbf{x}^T A\mathbf{x}}{\|\mathbf{x}\|^2} \leq \lambda_{\max}$ |
| Signature | $(\text{pos}, \text{neg}, \text{zero})$ is a congruence invariant |

---

## Quick Revision Questions

1. Write $Q = 4x^2 - 6xy + 9y^2$ in matrix form and classify it.

2. Find the principal axes and standard form of $Q = x^2 + 4xy + y^2$.

3. What is the maximum of $Q(\mathbf{x}) = 3x_1^2 + 5x_2^2$ on the unit circle?

4. State the Principal Axis Theorem and explain its geometric meaning.

5. A quadratic form has eigenvalues $2, -1, 3$. What is its signature? Is it positive definite?

6. Use Sylvester's criterion to determine if $A = \begin{pmatrix} 4 & 2 \\ 2 & 3 \end{pmatrix}$ is positive definite.

---

[← Previous: Orthogonal Matrices](03-orthogonal-matrices.md) · [Back to README](../README.md) · [Next: Unit 7 — Singular Value Decomposition →](../07-SVD-and-Applications/01-singular-value-decomposition.md)
