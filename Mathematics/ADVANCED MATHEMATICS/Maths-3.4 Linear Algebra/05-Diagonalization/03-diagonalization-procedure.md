# 5.3 Diagonalization Procedure

[← Previous: Similarity Transformation](02-similarity-transformation.md) · [Back to README](../README.md) · [Next: Applications — Matrix Powers →](04-applications-matrix-powers.md)

---

## Chapter Overview

This chapter gives a detailed, algorithmic approach to diagonalisation with comprehensive worked examples for $2 \times 2$ and $3 \times 3$ matrices, including all the row reduction steps.

---

## 1. Complete Algorithm

```
  ╔════════════════════════════════════════════════════╗
  ║           DIAGONALISATION ALGORITHM                ║
  ╠════════════════════════════════════════════════════╣
  ║                                                    ║
  ║  INPUT: n × n matrix A                             ║
  ║                                                    ║
  ║  1. Compute p(λ) = det(A − λI)                    ║
  ║  2. Factorise p(λ) → eigenvalues λ₁,...,λₖ        ║
  ║     with algebraic multiplicities am₁,...,amₖ      ║
  ║  3. For each λᵢ:                                  ║
  ║     a. Form A − λᵢI                               ║
  ║     b. Row reduce to find null space               ║
  ║     c. Compute gmᵢ = dim(ker(A − λᵢI))           ║
  ║     d. If gmᵢ < amᵢ → STOP: NOT diagonalisable   ║
  ║  4. Collect all eigenvectors → columns of P        ║
  ║  5. Form D = diag(λ₁,...,λₙ) matching column      ║
  ║     order of P                                     ║
  ║                                                    ║
  ║  OUTPUT: A = PDP⁻¹                                ║
  ╚════════════════════════════════════════════════════╝
```

---

## 2. Worked Example 1: $2 \times 2$

**Diagonalise** $A = \begin{pmatrix} 7 & 2 \\ -4 & 1 \end{pmatrix}$

**Step 1:** $p(\lambda) = (7-\lambda)(1-\lambda) + 8 = \lambda^2 - 8\lambda + 15 = (\lambda - 3)(\lambda - 5)$

**Step 2:** $\lambda_1 = 3$, $\lambda_2 = 5$, both $\text{am} = 1$

**Step 3a** ($\lambda = 3$):

$$A - 3I = \begin{pmatrix} 4 & 2 \\ -4 & -2 \end{pmatrix} \xrightarrow{R_2 + R_1} \begin{pmatrix} 4 & 2 \\ 0 & 0 \end{pmatrix} \xrightarrow{R_1/4} \begin{pmatrix} 1 & 1/2 \\ 0 & 0 \end{pmatrix}$$

$v_1 = -\frac{1}{2}v_2$, let $v_2 = 2$: $\mathbf{v}_1 = \begin{pmatrix} -1 \\ 2 \end{pmatrix}$

**Step 3b** ($\lambda = 5$):

$$A - 5I = \begin{pmatrix} 2 & 2 \\ -4 & -4 \end{pmatrix} \xrightarrow{\text{RREF}} \begin{pmatrix} 1 & 1 \\ 0 & 0 \end{pmatrix}$$

$v_1 = -v_2$, let $v_2 = 1$: $\mathbf{v}_2 = \begin{pmatrix} -1 \\ 1 \end{pmatrix}$

**Result:**

$$P = \begin{pmatrix} -1 & -1 \\ 2 & 1 \end{pmatrix}, \quad D = \begin{pmatrix} 3 & 0 \\ 0 & 5 \end{pmatrix}$$

$$A = PDP^{-1}$$

---

## 3. Worked Example 2: $3 \times 3$ (Distinct Eigenvalues)

**Diagonalise** $A = \begin{pmatrix} 1 & 2 & 0 \\ 0 & 3 & 0 \\ 2 & -4 & 2 \end{pmatrix}$

**Step 1:** Expand along row 2 or column 3:

$$p(\lambda) = (1-\lambda)(3-\lambda)(2-\lambda) - 0 + 0 = -(λ-1)(λ-2)(λ-3)$$

Wait — let me compute properly. Expanding $\det(A - \lambda I)$:

$$\det \begin{pmatrix} 1-\lambda & 2 & 0 \\ 0 & 3-\lambda & 0 \\ 2 & -4 & 2-\lambda \end{pmatrix}$$

Expand along column 3: $(2-\lambda) \det \begin{pmatrix} 1-\lambda & 2 \\ 0 & 3-\lambda \end{pmatrix} = (2-\lambda)(1-\lambda)(3-\lambda)$

Eigenvalues: $\lambda_1 = 1, \lambda_2 = 2, \lambda_3 = 3$ (all distinct → diagonalisable!)

**Step 3a** ($\lambda = 1$): $A - I = \begin{pmatrix} 0 & 2 & 0 \\ 0 & 2 & 0 \\ 2 & -4 & 1 \end{pmatrix} \xrightarrow{\text{RREF}} \begin{pmatrix} 1 & 0 & 1/2 \\ 0 & 1 & 0 \\ 0 & 0 & 0 \end{pmatrix}$

$\mathbf{v}_1 = \begin{pmatrix} -1/2 \\ 0 \\ 1 \end{pmatrix}$ or use $\begin{pmatrix} -1 \\ 0 \\ 2 \end{pmatrix}$

**Step 3b** ($\lambda = 2$): $A - 2I = \begin{pmatrix} -1 & 2 & 0 \\ 0 & 1 & 0 \\ 2 & -4 & 0 \end{pmatrix} \xrightarrow{\text{RREF}} \begin{pmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 0 \end{pmatrix}$

$\mathbf{v}_2 = \begin{pmatrix} 0 \\ 0 \\ 1 \end{pmatrix}$

**Step 3c** ($\lambda = 3$): $A - 3I = \begin{pmatrix} -2 & 2 & 0 \\ 0 & 0 & 0 \\ 2 & -4 & -1 \end{pmatrix} \xrightarrow{\text{RREF}} \begin{pmatrix} 1 & -1 & 0 \\ 0 & 0 & 1 \\ 0 & 0 & 0 \end{pmatrix}$

$v_1 = v_2, v_3 = 0$: $\mathbf{v}_3 = \begin{pmatrix} 1 \\ 1 \\ 0 \end{pmatrix}$

**Result:**

$$P = \begin{pmatrix} -1 & 0 & 1 \\ 0 & 0 & 1 \\ 2 & 1 & 0 \end{pmatrix}, \quad D = \begin{pmatrix} 1 & 0 & 0 \\ 0 & 2 & 0 \\ 0 & 0 & 3 \end{pmatrix}$$

---

## 4. Worked Example 3: Repeated Eigenvalue (Diagonalisable)

**Diagonalise** $A = \begin{pmatrix} 5 & 0 & 0 \\ 0 & 5 & 0 \\ 0 & 0 & 3 \end{pmatrix}$

$p(\lambda) = (5-\lambda)^2(3-\lambda)$

$\lambda = 5$: $\text{am} = 2$, $A - 5I = \begin{pmatrix} 0 & 0 & 0 \\ 0 & 0 & 0 \\ 0 & 0 & -2 \end{pmatrix}$, rank = 1, $\text{gm} = 2$ ✓

$E_5 = \text{span}\left\{\begin{pmatrix} 1 \\ 0 \\ 0 \end{pmatrix}, \begin{pmatrix} 0 \\ 1 \\ 0 \end{pmatrix}\right\}$

$\lambda = 3$: $\text{am} = 1, \text{gm} = 1$

$E_3 = \text{span}\left\{\begin{pmatrix} 0 \\ 0 \\ 1 \end{pmatrix}\right\}$

Total: 3 eigenvectors for $3 \times 3$ matrix. ✓ (This matrix was already diagonal!)

---

## 5. Worked Example 4: NOT Diagonalisable

**Attempt to diagonalise** $A = \begin{pmatrix} 1 & 1 & 0 \\ 0 & 1 & 1 \\ 0 & 0 & 1 \end{pmatrix}$

$p(\lambda) = (1-\lambda)^3$

$\lambda = 1$: $\text{am} = 3$

$$A - I = \begin{pmatrix} 0 & 1 & 0 \\ 0 & 0 & 1 \\ 0 & 0 & 0 \end{pmatrix}, \quad \text{rank} = 2, \quad \text{gm} = 3 - 2 = 1$$

Only 1 eigenvector! $\text{gm}(1) = 1 < 3 = \text{am}(1)$

**Conclusion:** NOT diagonalisable. ✗

---

## 6. Quick Test: Is Diagonalisation Possible?

```
  ┌────────────────────────────────────────────┐
  │       QUICK DIAGONALISABILITY TESTS         │
  ├────────────────────────────────────────────┤
  │                                            │
  │ ✓ Definitely diagonalisable:               │
  │   • n distinct eigenvalues                 │
  │   • Symmetric matrix                       │
  │   • Already diagonal                       │
  │                                            │
  │ ? Might be diagonalisable:                 │
  │   • Repeated eigenvalues — check gm = am   │
  │                                            │
  │ ✗ Definitely NOT diagonalisable:           │
  │   • Some gm < am                           │
  │   • Has non-trivial Jordan blocks          │
  └────────────────────────────────────────────┘
```

---

## Summary Table

| Step | Action |
|------|--------|
| 1 | Compute $\det(A - \lambda I) = 0$ → eigenvalues |
| 2 | Record algebraic multiplicities |
| 3 | For each $\lambda$: row reduce $A - \lambda I$, find null space |
| 4 | Check $\text{gm} = \text{am}$ for all eigenvalues |
| 5 | $P$ = matrix of eigenvectors (columns), $D$ = eigenvalues on diagonal |
| 6 | Verify $AP = PD$ |

---

## Quick Revision Questions

1. Diagonalise $A = \begin{pmatrix} 2 & 1 \\ 1 & 2 \end{pmatrix}$.

2. Show that $A = \begin{pmatrix} 0 & 1 \\ 0 & 0 \end{pmatrix}$ is not diagonalisable.

3. Diagonalise (if possible) $A = \begin{pmatrix} 1 & 0 & 0 \\ 0 & 2 & 1 \\ 0 & 0 & 2 \end{pmatrix}$.

4. If $A$ is diagonalisable and $B = P^{-1}AP = D$, is $B$ unique?

5. How many $3 \times 3$ matrices (up to similarity) have characteristic polynomial $(\lambda-1)(\lambda-2)^2$?

---

[← Previous: Similarity Transformation](02-similarity-transformation.md) · [Back to README](../README.md) · [Next: Applications — Matrix Powers →](04-applications-matrix-powers.md)
