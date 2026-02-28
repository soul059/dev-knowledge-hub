# 2.4 Matrix Representation of Linear Transformations

[← Previous: Rank-Nullity Theorem](03-rank-nullity-theorem.md) · [Back to README](../README.md) · [Next: Change of Basis →](05-change-of-basis.md)

---

## Chapter Overview

Every linear transformation between finite-dimensional vector spaces can be represented by a **matrix** once we fix bases for the domain and codomain. This chapter shows how to construct that matrix, how to use it for computation, and how the matrix depends on the chosen bases.

---

## 1. The Key Idea

> **Theorem.** Let $T : V \to W$ be a linear transformation where $\dim(V) = n$ and $\dim(W) = m$. Choose a basis $\mathcal{B} = \{\mathbf{v}_1, \dots, \mathbf{v}_n\}$ for $V$ and a basis $\mathcal{C} = \{\mathbf{w}_1, \dots, \mathbf{w}_m\}$ for $W$. Then there exists a unique $m \times n$ matrix $[T]_{\mathcal{B}}^{\mathcal{C}}$ such that:
>
> $$[T(\mathbf{v})]_{\mathcal{C}} = [T]_{\mathcal{B}}^{\mathcal{C}} \cdot [\mathbf{v}]_{\mathcal{B}}$$

```
  ┌───────────────────────────────────────────────────┐
  │                                                   │
  │   v ∈ V ─────── T ──────────▶ T(v) ∈ W           │
  │     │                            │                │
  │   [·]_B                        [·]_C             │
  │     │                            │                │
  │     ▼                            ▼                │
  │   [v]_B ∈ ℝⁿ ─── [T]^C_B ──▶ [T(v)]_C ∈ ℝᵐ     │
  │          (coords)  (matrix)    (coords)           │
  │                                                   │
  └───────────────────────────────────────────────────┘
```

---

## 2. Construction

The matrix $[T]_{\mathcal{B}}^{\mathcal{C}}$ is built column by column:

$$[T]_{\mathcal{B}}^{\mathcal{C}} = \Big[\; [T(\mathbf{v}_1)]_{\mathcal{C}} \;\Big|\; [T(\mathbf{v}_2)]_{\mathcal{C}} \;\Big|\; \cdots \;\Big|\; [T(\mathbf{v}_n)]_{\mathcal{C}} \;\Big]$$

**Recipe:**
1. Apply $T$ to each basis vector $\mathbf{v}_j$ of $V$.
2. Express $T(\mathbf{v}_j)$ as a linear combination of the basis $\mathcal{C}$ of $W$.
3. The coefficients form the $j$-th **column** of the matrix.

---

## 3. Worked Example 1 — Standard Bases

$T : \mathbb{R}^2 \to \mathbb{R}^3$ defined by $T(x, y) = (x + y,\; 2x,\; x - y)$.

Using standard bases $\mathcal{B} = \{\mathbf{e}_1, \mathbf{e}_2\}$ and $\mathcal{C} = \{\mathbf{e}_1, \mathbf{e}_2, \mathbf{e}_3\}$:

$$T(\mathbf{e}_1) = T(1,0) = (1, 2, 1) = 1\mathbf{e}_1 + 2\mathbf{e}_2 + 1\mathbf{e}_3$$
$$T(\mathbf{e}_2) = T(0,1) = (1, 0, -1) = 1\mathbf{e}_1 + 0\mathbf{e}_2 + (-1)\mathbf{e}_3$$

$$[T]_{\mathcal{B}}^{\mathcal{C}} = \begin{pmatrix} 1 & 1 \\ 2 & 0 \\ 1 & -1 \end{pmatrix}$$

**Verify:** $T(3, 2) = (5, 6, 1)$.

$$\begin{pmatrix}1&1\\2&0\\1&-1\end{pmatrix}\begin{pmatrix}3\\2\end{pmatrix} = \begin{pmatrix}5\\6\\1\end{pmatrix} \quad ✓$$

---

## 4. Worked Example 2 — Non-Standard Bases

$T : \mathbb{R}^2 \to \mathbb{R}^2$ defined by $T(x, y) = (x + y, x - y)$.

$\mathcal{B} = \{(1, 1), (1, -1)\}$, $\mathcal{C} = \{(1, 0), (0, 1)\}$ (standard).

$$T(1, 1) = (2, 0) = 2(1,0) + 0(0,1) \implies \text{column 1} = \begin{pmatrix}2\\0\end{pmatrix}$$

$$T(1, -1) = (0, 2) = 0(1,0) + 2(0,1) \implies \text{column 2} = \begin{pmatrix}0\\2\end{pmatrix}$$

$$[T]_{\mathcal{B}}^{\mathcal{C}} = \begin{pmatrix}2 & 0 \\ 0 & 2\end{pmatrix}$$

Notice: The basis $\mathcal{B}$ made $T$ look like a **scaling** — a diagonal matrix! The right basis can simplify the matrix representation.

---

## 5. Worked Example 3 — Differentiation

$D : P_2 \to P_1$ defined by $D(p) = p'$.

$\mathcal{B} = \{1, x, x^2\}$ (basis of $P_2$), $\mathcal{C} = \{1, x\}$ (basis of $P_1$).

$$D(1) = 0 = 0 \cdot 1 + 0 \cdot x \implies \text{col 1} = \begin{pmatrix}0\\0\end{pmatrix}$$
$$D(x) = 1 = 1 \cdot 1 + 0 \cdot x \implies \text{col 2} = \begin{pmatrix}1\\0\end{pmatrix}$$
$$D(x^2) = 2x = 0 \cdot 1 + 2 \cdot x \implies \text{col 3} = \begin{pmatrix}0\\2\end{pmatrix}$$

$$[D]_{\mathcal{B}}^{\mathcal{C}} = \begin{pmatrix}0 & 1 & 0 \\ 0 & 0 & 2\end{pmatrix}$$

**Verify:** $D(3 + 2x - x^2) = 2 - 2x$.

$$\begin{pmatrix}0&1&0\\0&0&2\end{pmatrix}\begin{pmatrix}3\\2\\-1\end{pmatrix} = \begin{pmatrix}2\\-2\end{pmatrix} \implies 2 \cdot 1 + (-2) \cdot x = 2 - 2x \quad ✓$$

---

## 6. Composition Corresponds to Matrix Multiplication

If $T : V \to W$ and $S : W \to U$ are linear, with bases $\mathcal{B}, \mathcal{C}, \mathcal{D}$ respectively:

$$[S \circ T]_{\mathcal{B}}^{\mathcal{D}} = [S]_{\mathcal{C}}^{\mathcal{D}} \cdot [T]_{\mathcal{B}}^{\mathcal{C}}$$

```
  V ───── T ─────▶ W ───── S ─────▶ U
  
  [v]_B ── [T]^C_B ──▶ [T(v)]_C ── [S]^D_C ──▶ [S(T(v))]_D
  
        = [S]^D_C · [T]^C_B · [v]_B
        = [S ∘ T]^D_B · [v]_B
```

---

## 7. Inverse Transformations

If $T : V \to V$ is an isomorphism (bijective), then $T^{-1}$ exists and:

$$[T^{-1}]_{\mathcal{C}}^{\mathcal{B}} = \left([T]_{\mathcal{B}}^{\mathcal{C}}\right)^{-1}$$

---

## 8. The Matrix Depends on the Basis

The **same** transformation can have very different matrices in different bases:

| Basis Choice | Matrix | Appearance |
|-------------|--------|------------|
| Standard basis | General matrix | May be complicated |
| Eigenbasis (if exists) | Diagonal matrix | Simplest possible |
| Jordan basis | Jordan form | Near-diagonal |

This motivates **change of basis** (next chapter) and **diagonalisation** (Unit 5).

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| $[T]_{\mathcal{B}}^{\mathcal{C}}$ | Matrix representation of $T$ w.r.t. bases $\mathcal{B}$, $\mathcal{C}$ |
| Column $j$ | Coordinates of $T(\mathbf{v}_j)$ in basis $\mathcal{C}$ |
| Standard bases | Matrix entries come directly from $T$'s formula |
| Composition | $[S \circ T] = [S] \cdot [T]$ (matrix multiplication) |
| Inverse | $[T^{-1}] = [T]^{-1}$ |
| Differentiation matrix | Encodes $D$ acting on polynomial basis |
| Choice of basis matters | Good basis → simpler matrix |

---

## Quick Revision Questions

1. Find the matrix of $T(x, y) = (2x - y, x + 3y)$ with respect to standard bases.

2. Find the matrix of $T : P_2 \to P_1$ defined by $T(p) = p'$ using bases $\{1, x, x^2\}$ and $\{1, x\}$.

3. If $[T]_{\mathcal{B}}^{\mathcal{C}} = \begin{pmatrix}1&2\\3&4\end{pmatrix}$ and $\mathcal{B} = \{(1,0),(0,1)\}$, what is $T(3,5)$?

4. If $S$ and $T$ are both from $\mathbb{R}^2 \to \mathbb{R}^2$, how is the matrix of $S \circ T$ related to the matrices of $S$ and $T$?

5. Can a $3 \times 2$ matrix represent a linear transformation from $\mathbb{R}^2$ to $\mathbb{R}^3$? From $\mathbb{R}^3$ to $\mathbb{R}^2$?

6. Why might you prefer a non-standard basis when representing a transformation?

---

[← Previous: Rank-Nullity Theorem](03-rank-nullity-theorem.md) · [Back to README](../README.md) · [Next: Change of Basis →](05-change-of-basis.md)
