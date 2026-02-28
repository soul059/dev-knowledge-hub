# 1.6 Basis and Dimension

[← Previous: Linear Independence](05-linear-independence.md) · [Back to README](../README.md) · [Next: Unit 2 — Definition and Examples of Linear Transformations →](../02-Linear-Transformations/01-definition-and-examples.md)

---

## Chapter Overview

A **basis** is the "right-sized" set of vectors for a vector space — it spans the space with no redundancy. The number of vectors in a basis is the **dimension**, which is an intrinsic property of the space. This chapter ties together span and linear independence into one of the most powerful concepts in Linear Algebra.

---

## 1. Definition of a Basis

**Definition.** A set $\mathcal{B} = \{\mathbf{v}_1, \mathbf{v}_2, \dots, \mathbf{v}_n\}$ is a **basis** for a vector space $V$ if:

1. $\mathcal{B}$ is **linearly independent**, AND
2. $\mathcal{B}$ **spans** $V$ (i.e., $\text{span}(\mathcal{B}) = V$).

```
  ┌──────────────────────────────────────────────────┐
  │                                                  │
  │   Spanning sets ──────────┐                      │
  │                           │                      │
  │                    ┌──────┴───────┐              │
  │                    │   BASIS      │              │
  │                    │ (independent │              │
  │                    │  AND spans)  │              │
  │                    └──────┬───────┘              │
  │                           │                      │
  │   Independent sets ───────┘                      │
  │                                                  │
  └──────────────────────────────────────────────────┘
  
  A basis sits at the intersection:
  • Remove a vector → loses spanning
  • Add a vector    → loses independence
```

---

## 2. Examples of Bases

### Standard Basis of $\mathbb{R}^n$

$$\mathcal{E} = \{\mathbf{e}_1, \mathbf{e}_2, \dots, \mathbf{e}_n\}$$

where $\mathbf{e}_i$ has a 1 in position $i$ and 0 elsewhere.

For $\mathbb{R}^3$:
$$\mathbf{e}_1 = (1,0,0), \quad \mathbf{e}_2 = (0,1,0), \quad \mathbf{e}_3 = (0,0,1)$$

```
        z
        ▲  e₃
        │
        │
        │
        ┼──────────▶ y
       ╱              e₂
      ╱
     ╱  e₁
    ▼
    x
```

### Standard Basis of $P_n(\mathbb{R})$

$$\mathcal{B} = \{1, x, x^2, \dots, x^n\}$$

Every polynomial of degree $\leq n$ is a unique linear combination of these.

### Standard Basis of $M_{2 \times 2}(\mathbb{R})$

$$\mathcal{B} = \left\{ \begin{pmatrix}1&0\\0&0\end{pmatrix},\; \begin{pmatrix}0&1\\0&0\end{pmatrix},\; \begin{pmatrix}0&0\\1&0\end{pmatrix},\; \begin{pmatrix}0&0\\0&1\end{pmatrix} \right\}$$

### Non-Standard Basis of $\mathbb{R}^2$

$\mathcal{B} = \{(1, 1), (1, -1)\}$

Independent: $(1,1)$ is not a multiple of $(1,-1)$.  
Spanning: $(a,b) = \frac{a+b}{2}(1,1) + \frac{a-b}{2}(1,-1)$. ✓

---

## 3. Unique Representation Theorem

> **Theorem.** If $\mathcal{B} = \{\mathbf{v}_1, \dots, \mathbf{v}_n\}$ is a basis for $V$, then every vector $\mathbf{v} \in V$ can be written as a linear combination of $\mathcal{B}$ in **exactly one way**.

$$\mathbf{v} = c_1 \mathbf{v}_1 + c_2 \mathbf{v}_2 + \cdots + c_n \mathbf{v}_n$$

The scalars $(c_1, c_2, \dots, c_n)$ are the **coordinates** of $\mathbf{v}$ with respect to $\mathcal{B}$, written:

$$[\mathbf{v}]_{\mathcal{B}} = \begin{pmatrix} c_1 \\ c_2 \\ \vdots \\ c_n \end{pmatrix}$$

**Proof of uniqueness:**  
Suppose $\mathbf{v} = \sum a_i \mathbf{v}_i = \sum b_i \mathbf{v}_i$. Then $\sum (a_i - b_i)\mathbf{v}_i = \mathbf{0}$. By independence, $a_i - b_i = 0$ for all $i$, so $a_i = b_i$.

---

## 4. Coordinate Vectors — Example

Let $\mathcal{B} = \{(1,1), (1,-1)\}$ be a basis for $\mathbb{R}^2$.

Find the coordinates of $\mathbf{v} = (3, 1)$ in basis $\mathcal{B}$:

$c_1(1,1) + c_2(1,-1) = (3,1)$

$$\begin{cases} c_1 + c_2 = 3 \\ c_1 - c_2 = 1 \end{cases} \implies c_1 = 2,\; c_2 = 1$$

$$[\mathbf{v}]_{\mathcal{B}} = \begin{pmatrix} 2 \\ 1 \end{pmatrix}$$

```
  Standard basis view:           Basis B view:
  
        y                              v₂ = (1,-1)
        ▲                              ▲
        │    •(3,1)                     │   •(3,1) has coords [2,1]_B
        │   ╱                           │  ╱
        │  ╱                            │ ╱
        │ ╱                             │╱
   ─────┼──────▶ x              ────────┼────────▶ v₁ = (1,1)
        │                              │
```

---

## 5. Definition of Dimension

**Definition.** The **dimension** of a vector space $V$, denoted $\dim(V)$, is the number of vectors in any basis of $V$.

This is well-defined because of:

> **Theorem (Invariance of Dimension).** Any two bases for a finite-dimensional vector space $V$ have the **same number** of vectors.

### Dimensions of Common Spaces

| Vector Space | Dimension | Standard Basis |
|-------------|-----------|----------------|
| $\mathbb{R}^n$ | $n$ | $\{\mathbf{e}_1, \dots, \mathbf{e}_n\}$ |
| $P_n(\mathbb{R})$ | $n + 1$ | $\{1, x, x^2, \dots, x^n\}$ |
| $M_{m \times n}(\mathbb{R})$ | $mn$ | $\{E_{ij}\}$ (matrix with 1 in position $(i,j)$, 0 elsewhere) |
| $\{\mathbf{0}\}$ | $0$ | $\varnothing$ (empty set) |
| $\mathbb{C}$ over $\mathbb{R}$ | $2$ | $\{1, i\}$ |
| $\mathbb{C}$ over $\mathbb{C}$ | $1$ | $\{1\}$ |

---

## 6. Key Theorems

### Theorem 1: Size of Independent / Spanning Sets

In a finite-dimensional space $V$ with $\dim(V) = n$:

| Fact | Statement |
|------|-----------|
| 1 | Any linearly independent set has $\leq n$ vectors |
| 2 | Any spanning set has $\geq n$ vectors |
| 3 | An independent set of exactly $n$ vectors is a basis |
| 4 | A spanning set of exactly $n$ vectors is a basis |

### Theorem 2: Extending to a Basis

> Any linearly independent set in a finite-dimensional space $V$ can be extended to a basis of $V$.

### Theorem 3: Reducing to a Basis

> Any spanning set for a finite-dimensional space $V$ can be reduced to a basis of $V$ by removing redundant vectors.

```
  Independent set ──extend──▶  BASIS  ◀──reduce── Spanning set
       (≤ n vectors)          (= n vectors)        (≥ n vectors)
```

---

## 7. Finding a Basis — Procedures

### Method 1: Basis for Column Space of $A$

1. Row-reduce $A$.
2. Identify **pivot columns**.
3. The corresponding columns of the *original* $A$ form a basis for $\text{Col}(A)$.

```
       Original A                    Row Echelon Form
  ┌                 ┐           ┌                 ┐
  │ 1  2  3  4      │           │ 1  2  0  1      │
  │ 2  4  7  9      │  ──▶     │ 0  0  1  1      │
  │ 1  2  4  5      │           │ 0  0  0  0      │
  └                 ┘           └                 ┘
       ↑     ↑                       ↑     ↑
     col 1  col 3                  pivot  pivot
  
  Basis for Col(A): {col₁, col₃ of original A}
```

### Method 2: Basis for Null Space of $A$

1. Solve $A\mathbf{x} = \mathbf{0}$.
2. Express the general solution in terms of free variables.
3. The vectors multiplying the free variables form a basis for $\text{Null}(A)$.

### Method 3: Basis for a Subspace Defined by Equations

**Example:** $W = \{(a,b,c) \mid a + b - c = 0\}$

Parametrize: $c = a + b$, so $(a, b, a+b) = a(1,0,1) + b(0,1,1)$.

Basis: $\{(1,0,1), (0,1,1)\}$, and $\dim(W) = 2$.

---

## 8. The Dimension Formula for Subspaces

$$\dim(W_1 + W_2) = \dim(W_1) + \dim(W_2) - \dim(W_1 \cap W_2)$$

For a direct sum: $\dim(W_1 \oplus W_2) = \dim(W_1) + \dim(W_2)$.

---

## 9. Rank and Nullity (Preview)

For a matrix $A \in M_{m \times n}$:

| Quantity | Definition | Equals |
|----------|-----------|--------|
| $\text{rank}(A)$ | $\dim(\text{Col}(A))$ | Number of pivot columns |
| $\text{nullity}(A)$ | $\dim(\text{Null}(A))$ | Number of free variables |

$$\text{rank}(A) + \text{nullity}(A) = n \quad \text{(number of columns)}$$

This is the **Rank-Nullity Theorem**, studied in depth in Unit 2.

---

## 10. Real-World Application: Degrees of Freedom

In mechanical engineering, the **dimension** of the configuration space of a mechanism equals its **degrees of freedom**.

| System | Configuration Space | Dimension (DOF) |
|--------|-------------------|-----------------|
| Particle in a plane | $\mathbb{R}^2$ | 2 |
| Rigid body in 3D | $\mathbb{R}^3 \times SO(3)$ | 6 |
| Robot arm with $k$ joints | Subset of $\mathbb{R}^k$ | $k$ (or fewer with constraints) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Basis | Linearly independent set that spans $V$ |
| Dimension | Number of vectors in any basis |
| Unique representation | Every $\mathbf{v} \in V$ has exactly one coordinate vector w.r.t. a basis |
| $\dim(\mathbb{R}^n) = n$ | Standard basis has $n$ vectors |
| $\dim(P_n) = n+1$ | Basis: $\{1, x, \dots, x^n\}$ |
| $\dim(M_{m \times n}) = mn$ | Basis: matrix units $E_{ij}$ |
| Extension theorem | Independent set → extend to basis |
| Reduction theorem | Spanning set → reduce to basis |
| Rank + Nullity = $n$ | Fundamental relation for matrices |

---

## Quick Revision Questions

1. Prove that $\{(1, 1, 0), (0, 1, 1), (1, 0, 1)\}$ is a basis for $\mathbb{R}^3$.

2. What is the dimension of the subspace $W = \{(a, b, c, d) \in \mathbb{R}^4 \mid a + d = 0,\; b = c\}$?

3. Find a basis for the null space of $A = \begin{pmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 7 & 8 & 9 \end{pmatrix}$.

4. If $\dim(V) = 5$ and $W$ is a subspace with $\dim(W) = 5$, what can you conclude?

5. Why can't $\{(1, 0), (0, 1), (2, 3)\}$ be a basis for $\mathbb{R}^2$?

6. Find the coordinate vector of $p(x) = 3 + 2x - x^2$ with respect to the basis $\{1, x, x^2\}$ of $P_2$.

---

[← Previous: Linear Independence](05-linear-independence.md) · [Back to README](../README.md) · [Next: Unit 2 — Definition and Examples of Linear Transformations →](../02-Linear-Transformations/01-definition-and-examples.md)
