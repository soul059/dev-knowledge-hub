# 8.4 Isomorphisms

[← Previous: Polynomial Spaces](03-polynomial-spaces.md) · [Back to README](../README.md)

---

## Chapter Overview

Isomorphisms formalise the intuition that two vector spaces can be "the same" even when their elements look completely different. This final chapter shows that **dimension is the only invariant** — every finite-dimensional real vector space of dimension $n$ is isomorphic to $\mathbb{R}^n$.

---

## 1. Definition

> **Isomorphism.** A linear transformation $T: V \to W$ is an **isomorphism** if it is both:
> 1. **Injective** (one-to-one): $\ker(T) = \{\mathbf{0}\}$
> 2. **Surjective** (onto): $\text{Im}(T) = W$
>
> We write $V \cong W$ and say $V$ and $W$ are **isomorphic**.

Equivalently, $T$ is an isomorphism iff $T$ is an **invertible** linear map.

```
  Isomorphism T: V → W

  V                       W
  ┌─────────┐    T    ┌─────────┐
  │ u₁  ●───┼────────►┼──● w₁  │
  │         │         │         │
  │ u₂  ●───┼────────►┼──● w₂  │   Every element matched
  │         │         │         │   exactly once (bijection)
  │ u₃  ●───┼────────►┼──● w₃  │
  │         │         │         │
  │ u₄  ●───┼────────►┼──● w₄  │
  └─────────┘  T⁻¹    └─────────┘
               ◄────────
```

---

## 2. Isomorphism Criterion

> **Theorem.** Let $T: V \to W$ be linear with $\dim(V) = \dim(W) = n$ (finite). Then TFAE:
> 1. $T$ is an isomorphism
> 2. $T$ is injective
> 3. $T$ is surjective
> 4. $\text{rank}(T) = n$
> 5. The matrix $[T]$ (w.r.t. any bases) is invertible

In the finite-dimensional case with equal dimensions, **injective $\Leftrightarrow$ surjective $\Leftrightarrow$ bijective**.

---

## 3. The Fundamental Classification Theorem

> **Theorem.** Two finite-dimensional real vector spaces are isomorphic **if and only if** they have the same dimension.
>
> $$V \cong W \iff \dim(V) = \dim(W)$$

**Proof sketch:**
- ($\Rightarrow$) Isomorphisms preserve linear independence and spanning, hence dimension.
- ($\Leftarrow$) If $\dim(V) = \dim(W) = n$, pick bases $\{v_1, \dots, v_n\}$ and $\{w_1, \dots, w_n\}$. Define $T(v_i) = w_i$ and extend linearly. Then $T$ is an isomorphism.

**Consequence:** Dimension is the **complete invariant** for finite-dimensional vector spaces.

```
  Classification of finite-dimensional real vector spaces:

     dim = 1:  ℝ ≅ P₀ ≅ {diagonal 1×1 matrices}
     dim = 2:  ℝ² ≅ P₁ ≅ ℂ (as ℝ-space) ≅ {solutions of y'' + y = 0}
     dim = 3:  ℝ³ ≅ P₂ ≅ {3×3 diagonal matrices}
     dim = 4:  ℝ⁴ ≅ P₃ ≅ M₂ₓ₂ ≅ {2×2 symmetric} ⊕ {2×2 skew-sym}
     dim = n:  ℝⁿ ≅ Pₙ₋₁ ≅ ... (all the same!)
```

---

## 4. The Coordinate Isomorphism

The most important isomorphism in linear algebra:

> **Theorem.** Let $\mathcal{B} = \{b_1, \dots, b_n\}$ be an ordered basis for $V$. The map
>
> $$\phi_{\mathcal{B}}: V \to \mathbb{R}^n, \quad \phi_{\mathcal{B}}(\mathbf{v}) = [\mathbf{v}]_{\mathcal{B}}$$
>
> is an isomorphism.

**This is why we can do linear algebra using matrices:** every problem in an abstract $n$-dimensional space reduces to a problem in $\mathbb{R}^n$.

```
  Coordinate isomorphism — the bridge between abstract and concrete:

  Abstract space V              Concrete space ℝⁿ
  ┌────────────────┐            ┌────────────────┐
  │                │   φ_B      │                │
  │   v = Σcᵢbᵢ  ─┼───────────►┼─  [v]_B = c   │
  │                │            │    (column     │
  │  T: V → V    ─┼───────────►┼─   [T]_B: ℝⁿ→ℝⁿ│
  │                │            │    (matrix)    │
  │  T(v)        ─┼───────────►┼─  [T]_B · c   │
  │                │   φ_B      │                │
  └────────────────┘            └────────────────┘
       abstract                    computable!
```

---

## 5. Properties Preserved by Isomorphisms

If $T: V \to W$ is an isomorphism:

| Property | Preserved? | Details |
|----------|:----------:|---------|
| Dimension | ✓ | $\dim(V) = \dim(W)$ |
| Linear independence | ✓ | $\{v_i\}$ indep. $\Leftrightarrow$ $\{Tv_i\}$ indep. |
| Spanning | ✓ | $\text{span}\{v_i\} = V$ $\Leftrightarrow$ $\text{span}\{Tv_i\} = W$ |
| Basis | ✓ | $\mathcal{B}$ basis of $V$ $\Leftrightarrow$ $T(\mathcal{B})$ basis of $W$ |
| Subspace structure | ✓ | $U \leq V$ $\Leftrightarrow$ $T(U) \leq W$ |
| Zero vector | ✓ | $T(\mathbf{0}_V) = \mathbf{0}_W$ |
| Coordinates | ✓ | Same coefficients in corresponding bases |

**Not preserved** (depends on extra structure):
- Norm, inner product (unless $T$ is also an **isometry**)
- Specific matrix entries

---

## 6. Isomorphism vs. Equality vs. Similarity

| Concept | Notation | Meaning |
|---------|:--------:|---------|
| Equal | $V = W$ | Same set, same operations |
| Isomorphic | $V \cong W$ | Same structure, possibly different elements |
| Similar (matrices) | $A \sim B$ | $B = P^{-1}AP$; same transformation, different bases |

**Similarity is isomorphism for matrices:** $A$ and $B$ represent the same linear map iff $A \sim B$.

---

## 7. Examples of Isomorphisms

### 7.1 $\mathcal{P}_2 \cong \mathbb{R}^3$

$$T(a + bx + cx^2) = \begin{pmatrix} a \\ b \\ c \end{pmatrix}$$

The coordinate map w.r.t. the standard basis.

### 7.2 $M_{2 \times 2} \cong \mathbb{R}^4$

$$T\begin{pmatrix} a & b \\ c & d \end{pmatrix} = \begin{pmatrix} a \\ b \\ c \\ d \end{pmatrix}$$

### 7.3 $\mathbb{C} \cong \mathbb{R}^2$ (as real vector spaces)

$$T(a + bi) = \begin{pmatrix} a \\ b \end{pmatrix}$$

### 7.4 Non-example

$\mathcal{P}_2 \not\cong \mathcal{P}_3$ because $\dim(\mathcal{P}_2) = 3 \neq 4 = \dim(\mathcal{P}_3)$.

---

## 8. The Isomorphism $\text{Hom}(V, W) \cong M_{m \times n}$

The space of all linear transformations $T: V \to W$ (where $\dim V = n$, $\dim W = m$) is itself a vector space of dimension $mn$, isomorphic to the space of $m \times n$ matrices:

$$\text{Hom}(V, W) \cong M_{m \times n}(\mathbb{R})$$

The isomorphism is given by choosing bases and taking matrix representations:

$$T \mapsto [T]_{\mathcal{B}_V, \mathcal{B}_W}$$

---

## 9. Dual Spaces (Preview)

The **dual space** $V^* = \text{Hom}(V, \mathbb{R})$ consists of all linear functionals $\varphi: V \to \mathbb{R}$.

- $\dim(V^*) = \dim(V)$, so $V^* \cong V$ (but **not** canonically for $\dim > 0$).
- With an inner product, we get the canonical isomorphism via Riesz Representation: $\varphi(\cdot) = \langle \mathbf{w}, \cdot \rangle$ for a unique $\mathbf{w}$.

**Dual basis:** If $\mathcal{B} = \{e_1, \dots, e_n\}$ is a basis for $V$, the dual basis $\{e^1, \dots, e^n\}$ for $V^*$ satisfies:

$$e^i(e_j) = \delta_{ij} = \begin{cases} 1 & i = j \\ 0 & i \neq j \end{cases}$$

---

## 10. The Big Picture

```
  ┌─────────────────────────────────────────────────────┐
  │         THE STRUCTURE OF LINEAR ALGEBRA              │
  │                                                     │
  │  ┌──────────┐     coordinate      ┌──────────┐     │
  │  │ Abstract │     isomorphism     │  ℝⁿ      │     │
  │  │ space V  │ ◄═══════════════► │ (concrete) │     │
  │  │ dim = n  │     φ_B            │           │     │
  │  └────┬─────┘                    └─────┬─────┘     │
  │       │ T: V → V                       │ [T]_B     │
  │       ▼                                ▼           │
  │  ┌──────────┐                    ┌──────────┐     │
  │  │ Abstract │     isomorphism     │  ℝⁿ      │     │
  │  │ space V  │ ◄═══════════════► │ (concrete) │     │
  │  └──────────┘                    └──────────┘     │
  │                                                     │
  │  Key insight: DIMENSION determines EVERYTHING       │
  │  (for finite-dimensional vector spaces over ℝ)      │
  └─────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| Isomorphism | Bijective linear map; $T$ and $T^{-1}$ both linear |
| Criterion | Same $\dim$ $\Leftrightarrow$ isomorphic (finite-dim) |
| Classification | All $n$-dim real spaces $\cong \mathbb{R}^n$ |
| Coordinate map | $\phi_{\mathcal{B}}: V \xrightarrow{\sim} \mathbb{R}^n$; the fundamental bridge |
| Preserved properties | Dimension, independence, spanning, subspaces |
| $\text{Hom}(V,W)$ | $\cong M_{m \times n}$; $\dim = mn$ |
| Dual space | $V^* \cong V$ (non-canonically); $\dim(V^*) = \dim(V)$ |

---

## Quick Revision Questions

1. Prove that $T: \mathcal{P}_1 \to \mathbb{R}^2$ defined by $T(a + bx) = (a+b,\; a-b)^T$ is an isomorphism.

2. Are the spaces $M_{2 \times 3}(\mathbb{R})$ and $\mathcal{P}_5(\mathbb{R})$ isomorphic? Justify.

3. Let $V$ have basis $\{v_1, v_2, v_3\}$ and $T: V \to V$ satisfy $T(v_1) = v_2$, $T(v_2) = v_3$, $T(v_3) = v_1$. Find the matrix of $T$ and determine if $T$ is an isomorphism.

4. State the dual basis of $\{1, x, x^2\}$ in $\mathcal{P}_2^*$ and verify $e^i(e_j) = \delta_{ij}$.

5. Explain why $\mathbb{R}^4 \cong M_{2 \times 2}(\mathbb{R})$ but an inner product on $\mathbb{R}^4$ does **not** automatically transfer to $M_{2 \times 2}$.

---

[← Previous: Polynomial Spaces](03-polynomial-spaces.md) · [Back to README](../README.md)
