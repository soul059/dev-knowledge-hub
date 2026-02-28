# 4.3 Finding Eigenvalues and Eigenvectors

[← Previous: Characteristic Polynomial](02-characteristic-polynomial.md) · [Back to README](../README.md) · [Next: Properties of Eigenvalues →](04-properties.md)

---

## Chapter Overview

This chapter provides a systematic, step-by-step procedure for computing eigenvalues and eigenvectors, with detailed worked examples for $2 \times 2$ and $3 \times 3$ matrices.

---

## 1. The Complete Procedure

```
  ┌─────────────────────────────┐
  │  Step 1: Form A − λI        │
  └──────────────┬──────────────┘
                 ▼
  ┌─────────────────────────────┐
  │  Step 2: Compute det(A−λI)  │
  │  → characteristic polynomial│
  └──────────────┬──────────────┘
                 ▼
  ┌─────────────────────────────┐
  │  Step 3: Solve p(λ) = 0     │
  │  → eigenvalues λ₁, λ₂, ... │
  └──────────────┬──────────────┘
                 ▼
  ┌─────────────────────────────┐
  │  Step 4: For each λᵢ, solve │
  │  (A − λᵢI)v = 0             │
  │  → eigenvectors              │
  └──────────────┬──────────────┘
                 ▼
  ┌─────────────────────────────┐
  │  Step 5: Write eigenspaces  │
  │  E_λᵢ = span{...}           │
  └─────────────────────────────┘
```

---

## 2. Worked Example: $2 \times 2$ Matrix

Find the eigenvalues and eigenvectors of:

$$A = \begin{pmatrix} 6 & -1 \\ 2 & 3 \end{pmatrix}$$

**Step 1:** $A - \lambda I = \begin{pmatrix} 6-\lambda & -1 \\ 2 & 3-\lambda \end{pmatrix}$

**Step 2:** $\det(A - \lambda I) = (6-\lambda)(3-\lambda) + 2 = \lambda^2 - 9\lambda + 20$

**Step 3:** $\lambda^2 - 9\lambda + 20 = (\lambda - 4)(\lambda - 5) = 0$

$\lambda_1 = 4, \quad \lambda_2 = 5$

**Step 4a** ($\lambda_1 = 4$):

$$(A - 4I)\mathbf{v} = \begin{pmatrix} 2 & -1 \\ 2 & -1 \end{pmatrix}\begin{pmatrix} v_1 \\ v_2 \end{pmatrix} = \mathbf{0}$$

$2v_1 - v_2 = 0 \implies v_2 = 2v_1$

$$\mathbf{v}_1 = t\begin{pmatrix} 1 \\ 2 \end{pmatrix}, \quad E_4 = \text{span}\left\{\begin{pmatrix} 1 \\ 2 \end{pmatrix}\right\}$$

**Step 4b** ($\lambda_2 = 5$):

$$(A - 5I)\mathbf{v} = \begin{pmatrix} 1 & -1 \\ 2 & -2 \end{pmatrix}\begin{pmatrix} v_1 \\ v_2 \end{pmatrix} = \mathbf{0}$$

$v_1 - v_2 = 0 \implies v_1 = v_2$

$$\mathbf{v}_2 = t\begin{pmatrix} 1 \\ 1 \end{pmatrix}, \quad E_5 = \text{span}\left\{\begin{pmatrix} 1 \\ 1 \end{pmatrix}\right\}$$

---

## 3. Worked Example: $3 \times 3$ Matrix

Find eigenvalues and eigenvectors of:

$$A = \begin{pmatrix} 1 & 1 & 0 \\ 0 & 2 & 0 \\ 0 & 0 & 3 \end{pmatrix}$$

**Step 1-2:** Since $A$ is upper triangular:

$$p(\lambda) = (1-\lambda)(2-\lambda)(3-\lambda) = 0$$

**Step 3:** $\lambda_1 = 1, \lambda_2 = 2, \lambda_3 = 3$

**Step 4a** ($\lambda = 1$):

$$(A - I)\mathbf{v} = \begin{pmatrix} 0 & 1 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 2 \end{pmatrix}\mathbf{v} = \mathbf{0} \implies v_2 = 0, v_3 = 0$$

$$E_1 = \text{span}\left\{\begin{pmatrix} 1 \\ 0 \\ 0 \end{pmatrix}\right\}$$

**Step 4b** ($\lambda = 2$):

$$(A - 2I)\mathbf{v} = \begin{pmatrix} -1 & 1 & 0 \\ 0 & 0 & 0 \\ 0 & 0 & 1 \end{pmatrix}\mathbf{v} = \mathbf{0} \implies v_1 = v_2, v_3 = 0$$

$$E_2 = \text{span}\left\{\begin{pmatrix} 1 \\ 1 \\ 0 \end{pmatrix}\right\}$$

**Step 4c** ($\lambda = 3$):

$$(A - 3I)\mathbf{v} = \begin{pmatrix} -2 & 1 & 0 \\ 0 & -1 & 0 \\ 0 & 0 & 0 \end{pmatrix}\mathbf{v} = \mathbf{0} \implies v_2 = 0, v_1 = 0$$

$$E_3 = \text{span}\left\{\begin{pmatrix} 0 \\ 0 \\ 1 \end{pmatrix}\right\}$$

---

## 4. Worked Example: Repeated Eigenvalue

$$A = \begin{pmatrix} 3 & 1 \\ 0 & 3 \end{pmatrix}$$

$p(\lambda) = (3 - \lambda)^2 = 0 \implies \lambda = 3$ (algebraic multiplicity 2)

$$(A - 3I) = \begin{pmatrix} 0 & 1 \\ 0 & 0 \end{pmatrix}$$

Null space: $v_2 = 0$, $v_1$ free → $E_3 = \text{span}\left\{\begin{pmatrix} 1 \\ 0 \end{pmatrix}\right\}$

Geometric multiplicity = $\dim(E_3) = 1 < 2$ = algebraic multiplicity.

Only **one independent eigenvector** despite a double eigenvalue. This matrix is **not diagonalisable**.

---

## 5. Worked Example: Complex Eigenvalues

$$A = \begin{pmatrix} 1 & -2 \\ 1 & 3 \end{pmatrix}$$

$p(\lambda) = \lambda^2 - 4\lambda + 5 = 0$

$\lambda = \frac{4 \pm \sqrt{16 - 20}}{2} = \frac{4 \pm 2i}{2} = 2 \pm i$

For $\lambda = 2 + i$:

$$(A - (2+i)I) = \begin{pmatrix} -1-i & -2 \\ 1 & 1-i \end{pmatrix}$$

From row 1: $(-1-i)v_1 = 2v_2$, so $v_1 = \frac{2}{-(1+i)} = \frac{-2(1-i)}{2} = -(1-i) = -1 + i$

$$\mathbf{v} = \begin{pmatrix} -1+i \\ 1 \end{pmatrix}$$

Complex eigenvalues always come in conjugate pairs for real matrices.

---

## 6. Shortcuts and Checks

### For $2 \times 2$ Matrices

Given $A = \begin{pmatrix} a & b \\ c & d \end{pmatrix}$:

| Quantity | Formula |
|----------|--------|
| $\lambda_1 + \lambda_2$ | $a + d = \text{tr}(A)$ |
| $\lambda_1 \cdot \lambda_2$ | $ad - bc = \det(A)$ |
| Discriminant | $\Delta = (a-d)^2 + 4bc$ |

- $\Delta > 0$: two distinct real eigenvalues
- $\Delta = 0$: one repeated real eigenvalue
- $\Delta < 0$: two complex conjugate eigenvalues

### Verification Checklist

After finding eigenvalues and eigenvectors, always verify:

1. $A\mathbf{v} = \lambda \mathbf{v}$ (multiply and check)
2. $\sum \lambda_i = \text{tr}(A)$ (quick check)
3. $\prod \lambda_i = \det(A)$ (quick check)

---

## 7. Common Pitfalls

```
  ┌─────────────────────────────────────────────────┐
  │                  COMMON MISTAKES                  │
  ├─────────────────────────────────────────────────┤
  │ ✗ Forgetting v ≠ 0 in the eigenvector check     │
  │ ✗ Sign errors in det(A − λI) expansion          │
  │ ✗ Not row-reducing completely for eigenvectors   │
  │ ✗ Confusing algebraic and geometric multiplicity │
  │ ✗ Missing complex eigenvalues of real matrices   │
  └─────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Procedure | Form $A - \lambda I$ → det → solve → null space |
| $2 \times 2$ shortcut | $p(\lambda) = \lambda^2 - \text{tr}(A)\lambda + \det(A)$ |
| Triangular matrix | Eigenvalues read from diagonal |
| Repeated eigenvalue | May or may not give enough eigenvectors |
| Complex eigenvalues | Conjugate pairs for real matrices |
| Verification | Check $A\mathbf{v} = \lambda \mathbf{v}$, trace, determinant |

---

## Quick Revision Questions

1. Find all eigenvalues and eigenvectors of $A = \begin{pmatrix} 5 & 4 \\ 1 & 2 \end{pmatrix}$.

2. Find eigenvalues of the upper triangular matrix $A = \begin{pmatrix} 1 & 3 & 5 \\ 0 & 2 & 7 \\ 0 & 0 & 4 \end{pmatrix}$.

3. Show that $A = \begin{pmatrix} 0 & -1 \\ 1 & 0 \end{pmatrix}$ has no real eigenvalues.

4. Find eigenvalues and eigenvectors of $A = \begin{pmatrix} 2 & 0 \\ 0 & 2 \end{pmatrix}$. What is the geometric multiplicity?

5. A $3 \times 3$ matrix has $\text{tr}(A) = 6$ and eigenvalues $\lambda_1 = 1, \lambda_2 = 2$. Find $\lambda_3$ and $\det(A)$.

---

[← Previous: Characteristic Polynomial](02-characteristic-polynomial.md) · [Back to README](../README.md) · [Next: Properties of Eigenvalues →](04-properties.md)
