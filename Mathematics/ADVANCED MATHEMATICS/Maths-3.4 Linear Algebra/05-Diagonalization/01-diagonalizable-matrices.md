# 5.1 Diagonalizable Matrices

[← Previous: Unit 4 — Algebraic & Geometric Multiplicity](../04-Eigenvalues-and-Eigenvectors/05-algebraic-geometric-multiplicity.md) · [Back to README](../README.md) · [Next: Similarity Transformation →](02-similarity-transformation.md)

---

## Chapter Overview

A matrix is **diagonalisable** if it is similar to a diagonal matrix. This is one of the most powerful ideas in linear algebra — it reduces complicated matrix computations to simple operations on diagonal entries. This chapter establishes the conditions for diagonalisability.

---

## 1. Definition

> **Definition.** An $n \times n$ matrix $A$ is **diagonalisable** if there exists an invertible matrix $P$ such that:
>
> $$P^{-1}AP = D$$
>
> where $D = \text{diag}(\lambda_1, \lambda_2, \dots, \lambda_n)$ is diagonal.

Equivalently: $A = PDP^{-1}$.

The columns of $P$ are eigenvectors of $A$, and the diagonal entries of $D$ are the corresponding eigenvalues.

---

## 2. Why Diagonalisation Matters

```
  Without diagonalisation:          With diagonalisation:
  
  A¹⁰⁰ = A·A·A·...·A (100 times)  A¹⁰⁰ = P D¹⁰⁰ P⁻¹
                                         = P diag(λ₁¹⁰⁰,...,λₙ¹⁰⁰) P⁻¹
  
  Cost: O(n³ × 100)                Cost: O(n²) after setup
```

| Application | How diagonalisation helps |
|---|---|
| Matrix powers $A^k$ | $A^k = PD^kP^{-1}$; $D^k = \text{diag}(\lambda_1^k, \dots, \lambda_n^k)$ |
| Matrix exponential $e^A$ | $e^A = Pe^DP^{-1}$; $e^D = \text{diag}(e^{\lambda_1}, \dots, e^{\lambda_n})$ |
| Systems of ODEs | Decouples $\mathbf{x}' = A\mathbf{x}$ into independent equations |
| Recurrence relations | Solves $a_{n+1} = Aa_n$ in closed form |
| Understanding geometry | Reveals invariant directions and scaling factors |

---

## 3. The Diagonalisation Theorem

> **Theorem.** The following are equivalent for an $n \times n$ matrix $A$:
>
> 1. $A$ is diagonalisable
> 2. $A$ has $n$ linearly independent eigenvectors
> 3. $\text{gm}(\lambda_i) = \text{am}(\lambda_i)$ for every eigenvalue $\lambda_i$
> 4. $\sum_{i} \text{gm}(\lambda_i) = n$
> 5. $\mathbb{R}^n = E_{\lambda_1} \oplus E_{\lambda_2} \oplus \dots \oplus E_{\lambda_k}$ (direct sum of eigenspaces)

**Sufficient condition:** If $A$ has $n$ distinct eigenvalues, then $A$ is diagonalisable.

(The converse is false: $I$ has only eigenvalue 1 but is diagonalisable.)

---

## 4. Constructing the Diagonalisation

**Step-by-step procedure:**

1. Find all eigenvalues $\lambda_1, \dots, \lambda_k$ of $A$
2. For each $\lambda_i$, find a basis for $E_{\lambda_i} = \ker(A - \lambda_i I)$
3. Check: total number of basis vectors = $n$? If no → not diagonalisable.
4. Form $P$ by placing all eigenvectors as columns
5. Form $D$ with corresponding eigenvalues on the diagonal

```
  P = [v₁ | v₂ | ··· | vₙ]     D = diag(λ₁, λ₂, ..., λₙ)
  
  ┌──────────────────┐          ┌──────────────────┐
  │  ↑   ↑       ↑   │          │ λ₁  0  ···  0    │
  │  v₁  v₂ ··· vₙ   │          │  0  λ₂ ···  0    │
  │  ↓   ↓       ↓   │          │  ⋮       ⋱   ⋮   │
  └──────────────────┘          │  0   0  ··· λₙ   │
                                └──────────────────┘
  
  IMPORTANT: Column order in P must match diagonal order in D!
```

---

## 5. Worked Example

Diagonalise $A = \begin{pmatrix} 4 & 1 \\ 2 & 3 \end{pmatrix}$.

**Step 1:** $p(\lambda) = \lambda^2 - 7\lambda + 10 = (\lambda - 5)(\lambda - 2)$

Eigenvalues: $\lambda_1 = 5$, $\lambda_2 = 2$

**Step 2:**

$\lambda_1 = 5$: $(A - 5I) = \begin{pmatrix} -1 & 1 \\ 2 & -2 \end{pmatrix} \implies \mathbf{v}_1 = \begin{pmatrix} 1 \\ 1 \end{pmatrix}$

$\lambda_2 = 2$: $(A - 2I) = \begin{pmatrix} 2 & 1 \\ 2 & 1 \end{pmatrix} \implies \mathbf{v}_2 = \begin{pmatrix} 1 \\ -2 \end{pmatrix}$

**Step 3:** 2 eigenvectors for $2 \times 2$ matrix ✓

**Step 4-5:**

$$P = \begin{pmatrix} 1 & 1 \\ 1 & -2 \end{pmatrix}, \quad D = \begin{pmatrix} 5 & 0 \\ 0 & 2 \end{pmatrix}$$

**Verification:** $AP = PD$

$$AP = \begin{pmatrix} 4 & 1 \\ 2 & 3 \end{pmatrix}\begin{pmatrix} 1 & 1 \\ 1 & -2 \end{pmatrix} = \begin{pmatrix} 5 & 2 \\ 5 & -4 \end{pmatrix}$$

$$PD = \begin{pmatrix} 1 & 1 \\ 1 & -2 \end{pmatrix}\begin{pmatrix} 5 & 0 \\ 0 & 2 \end{pmatrix} = \begin{pmatrix} 5 & 2 \\ 5 & -4 \end{pmatrix} \quad ✓$$

---

## 6. Non-Diagonalisable Example

$$A = \begin{pmatrix} 2 & 1 \\ 0 & 2 \end{pmatrix}$$

$p(\lambda) = (2 - \lambda)^2$, so $\lambda = 2$ with $\text{am} = 2$.

$A - 2I = \begin{pmatrix} 0 & 1 \\ 0 & 0 \end{pmatrix}$, rank = 1, so $\text{gm} = 2 - 1 = 1$.

Only one eigenvector: $\mathbf{v} = (1, 0)^T$.

Cannot form invertible $P$ — **not diagonalisable**.

This is a **Jordan block** $J_2(2) = \begin{pmatrix} 2 & 1 \\ 0 & 2 \end{pmatrix}$.

---

## 7. Classes of Always-Diagonalisable Matrices

| Matrix class | Why always diagonalisable |
|---|---|
| Real symmetric ($A = A^T$) | Spectral theorem guarantees it |
| Hermitian ($A = A^*$) | Complex version of spectral theorem |
| Normal ($A^*A = AA^*$) | Generalisation of symmetric/Hermitian |
| Distinct eigenvalues | Automatic linear independence |
| Diagonal matrices | Already diagonal |
| Orthogonal ($A^TA = I$) | Normal, so diagonalisable (over $\mathbb{C}$) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Diagonalisable | $A = PDP^{-1}$, $P$ = eigenvectors, $D$ = eigenvalues |
| Criterion | Need $n$ linearly independent eigenvectors |
| Equivalent | $\text{gm}(\lambda) = \text{am}(\lambda)$ for all $\lambda$ |
| Sufficient | $n$ distinct eigenvalues → diagonalisable |
| Symmetric | Always diagonalisable |
| $A^k$ | $= PD^kP^{-1}$ (compute powers easily) |
| Not diagonalisable | Jordan form is the next best thing |

---

## Quick Revision Questions

1. State the diagonalisation theorem (three equivalent conditions).

2. Diagonalise $A = \begin{pmatrix} 3 & 1 \\ 0 & 2 \end{pmatrix}$.

3. Why is $A = \begin{pmatrix} 5 & 1 \\ 0 & 5 \end{pmatrix}$ not diagonalisable?

4. Show that if $A$ is diagonalisable, so is $A^T$.

5. If $A = PDP^{-1}$, express $A^{10}$ in terms of $P$ and $D$.

---

[← Previous: Unit 4 — Algebraic & Geometric Multiplicity](../04-Eigenvalues-and-Eigenvectors/05-algebraic-geometric-multiplicity.md) · [Back to README](../README.md) · [Next: Similarity Transformation →](02-similarity-transformation.md)
