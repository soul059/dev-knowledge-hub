# 2.3 Rank-Nullity Theorem

[← Previous: Kernel and Range](02-kernel-and-range.md) · [Back to README](../README.md) · [Next: Matrix Representation →](04-matrix-representation.md)

---

## Chapter Overview

The **Rank-Nullity Theorem** is one of the central results of Linear Algebra. It states that for any linear transformation $T : V \to W$, the dimension of the domain is split between what $T$ "kills" (kernel) and what $T$ "produces" (image). This chapter states the theorem, proves it, and explores its many consequences.

---

## 1. Statement

> **Rank-Nullity Theorem.** Let $T : V \to W$ be a linear transformation where $V$ is finite-dimensional. Then:
>
> $$\dim(V) = \dim(\ker(T)) + \dim(\text{Im}(T))$$
>
> Or equivalently: **nullity($T$) + rank($T$) = dim($V$)**

Where:
- **nullity($T$)** $= \dim(\ker(T))$
- **rank($T$)** $= \dim(\text{Im}(T))$

```
  ┌──────────────────────────────────────────────┐
  │                                              │
  │        dim(V) = nullity(T) + rank(T)         │
  │                                              │
  │   ┌────────────── dim(V) ──────────────┐     │
  │   │                                    │     │
  │   │   nullity(T)   │     rank(T)       │     │
  │   │   dim(ker(T))  │   dim(Im(T))      │     │
  │   │   "lost"       │   "preserved"     │     │
  │   └────────────────┴───────────────────┘     │
  │                                              │
  └──────────────────────────────────────────────┘
```

---

## 2. Proof

Let $\dim(V) = n$, $\dim(\ker(T)) = k$.

**Step 1:** Let $\{\mathbf{u}_1, \dots, \mathbf{u}_k\}$ be a basis for $\ker(T)$.

**Step 2:** Extend to a basis for $V$: $\{\mathbf{u}_1, \dots, \mathbf{u}_k, \mathbf{v}_1, \dots, \mathbf{v}_r\}$ where $r = n - k$.

**Step 3:** Claim: $\{T(\mathbf{v}_1), \dots, T(\mathbf{v}_r)\}$ is a basis for $\text{Im}(T)$.

**(a) Spanning:** Any $\mathbf{w} \in \text{Im}(T)$ equals $T(\mathbf{x})$ for some $\mathbf{x} = \sum a_i \mathbf{u}_i + \sum b_j \mathbf{v}_j$.

$$T(\mathbf{x}) = \sum a_i \underbrace{T(\mathbf{u}_i)}_{=\mathbf{0}} + \sum b_j T(\mathbf{v}_j) = \sum b_j T(\mathbf{v}_j)$$

**(b) Independence:** Suppose $\sum c_j T(\mathbf{v}_j) = \mathbf{0}$. Then $T(\sum c_j \mathbf{v}_j) = \mathbf{0}$, so $\sum c_j \mathbf{v}_j \in \ker(T)$.

Thus $\sum c_j \mathbf{v}_j = \sum d_i \mathbf{u}_i$, giving $\sum d_i \mathbf{u}_i - \sum c_j \mathbf{v}_j = \mathbf{0}$.

Since $\{\mathbf{u}_1, \dots, \mathbf{u}_k, \mathbf{v}_1, \dots, \mathbf{v}_r\}$ is a basis, all coefficients are 0. In particular, $c_j = 0$ for all $j$.

**Conclusion:** $\dim(\text{Im}(T)) = r = n - k$. ∎

---

## 3. For Matrices: Rank + Nullity = Number of Columns

For $A \in M_{m \times n}$ and $T(\mathbf{x}) = A\mathbf{x}$:

$$\text{rank}(A) + \text{nullity}(A) = n$$

| Quantity | Equals | Found By |
|----------|--------|----------|
| $n$ | Number of columns | Given |
| $\text{rank}(A)$ | Number of pivot columns | Row-reduce |
| $\text{nullity}(A)$ | Number of free variables | $n - \text{rank}$ |

### Example

$$A = \begin{pmatrix} 1 & 3 & 5 & 7 \\ 2 & 6 & 10 & 14 \\ 1 & 3 & 6 & 9 \end{pmatrix}$$

```
  Row reduce:
  ┌                   ┐       ┌                   ┐
  │ 1   3   5   7     │       │ 1   3   5   7     │
  │ 2   6  10  14     │ ────▶ │ 0   0   1   2     │
  │ 1   3   6   9     │       │ 0   0   0   0     │
  └                   ┘       └                   ┘
  
  Pivots: columns 1, 3  →  rank = 2
  Free: columns 2, 4    →  nullity = 2
  
  Check: rank + nullity = 2 + 2 = 4 = n ✓
```

---

## 4. Consequences and Corollaries

### Corollary 1: Injective ⟺ Surjective (Same-Dimensional Spaces)

If $\dim(V) = \dim(W) = n$ and $T : V \to W$ is linear:

$$T \text{ is injective} \iff T \text{ is surjective} \iff T \text{ is bijective}$$

**Proof:** $T$ injective $\iff$ nullity = 0 $\iff$ rank = $n$ = $\dim(W)$ $\iff$ $T$ surjective.

### Corollary 2: Dimension Constraints

| Condition | Consequence |
|-----------|------------|
| $\dim(V) > \dim(W)$ | $T$ cannot be injective (ker must be non-trivial) |
| $\dim(V) < \dim(W)$ | $T$ cannot be surjective (Im cannot fill $W$) |
| $\dim(V) = \dim(W)$ | Injective ⟺ Surjective ⟺ Bijective |

```
  dim(V) > dim(W):                 dim(V) < dim(W):
  
  V ─────▶ W                      V ─────▶ W
  [● ● ● ●]  [● ● ●]             [● ●]     [● ● ● ●]
  
  Cannot be 1-1                    Cannot be onto
  (pigeonhole: ker ≠ {0})         (not enough "material")
```

### Corollary 3: Invertible Matrix Theorem (Additional Conditions)

For an $n \times n$ matrix $A$, the following are equivalent:

| # | Condition |
|---|-----------|
| 1 | $A$ is invertible |
| 2 | $\text{rank}(A) = n$ |
| 3 | $\text{nullity}(A) = 0$ |
| 4 | $\ker(A) = \{\mathbf{0}\}$ |
| 5 | Columns of $A$ are linearly independent |
| 6 | Columns of $A$ span $\mathbb{R}^n$ |
| 7 | $\det(A) \neq 0$ |
| 8 | $A\mathbf{x} = \mathbf{b}$ has a unique solution for every $\mathbf{b}$ |

---

## 5. Applications of Rank-Nullity

### Application 1: Counting Solutions

For $A\mathbf{x} = \mathbf{b}$ (where $A$ is $m \times n$):

| Rank vs. $m$ and $n$ | Solutions to $A\mathbf{x} = \mathbf{b}$ |
|----------------------|----------------------------------------|
| rank = $n$ = $m$ | Unique solution (if consistent) |
| rank = $n$ < $m$ | At most one solution |
| rank < $n$, rank = $m$ | Infinitely many (if consistent) |
| rank < $n$, rank < $m$ | Infinitely many or none |

### Application 2: Existence of Non-Trivial Solutions

If $A$ is $m \times n$ with $n > m$, then $\text{nullity}(A) \geq n - m > 0$, so $A\mathbf{x} = \mathbf{0}$ always has a non-trivial solution.

> **Principle:** A homogeneous system with more unknowns than equations always has a non-trivial solution.

### Application 3: Rank of Transpose

$$\text{rank}(A) = \text{rank}(A^T)$$

The row rank equals the column rank.

---

## 6. Dimension Diagram

```
                    dim(V) = n
         ┌──────────────┴──────────────┐
         │                             │
    nullity(T) = k              rank(T) = n - k
    dim(ker(T))                 dim(Im(T))
         │                             │
         │                             │
    ker(T) ⊆ V                  Im(T) ⊆ W
    subspace                     subspace
```

---

## 7. Worked Example — Full Analysis

Let $T : \mathbb{R}^4 \to \mathbb{R}^3$ be defined by $T(\mathbf{x}) = A\mathbf{x}$ where:

$$A = \begin{pmatrix} 1 & 0 & 2 & 1 \\ 0 & 1 & 1 & 0 \\ 1 & 1 & 3 & 1 \end{pmatrix}$$

**Step 1: Row-reduce**
```
  ┌                  ┐        ┌                  ┐
  │ 1  0  2  1       │  R₃-R₁ │ 1  0  2  1       │
  │ 0  1  1  0       │ ──────▶│ 0  1  1  0       │
  │ 1  1  3  1       │  R₃-R₂ │ 0  0  0  0       │
  └                  ┘        └                  ┘
```

**Step 2: Read off**
- Pivots: columns 1, 2 → rank = 2
- Free variables: $x_3, x_4$ → nullity = 2
- Check: $2 + 2 = 4$ = number of columns ✓

**Step 3: Kernel**
$x_1 = -2x_3 - x_4$, $x_2 = -x_3$, $x_3, x_4$ free.

$$\ker(T) = \text{span}\left\{\begin{pmatrix}-2\\-1\\1\\0\end{pmatrix}, \begin{pmatrix}-1\\0\\0\\1\end{pmatrix}\right\}$$

**Step 4: Range**
$\text{Im}(T) = \text{Col}(A) = \text{span}\left\{\begin{pmatrix}1\\0\\1\end{pmatrix}, \begin{pmatrix}0\\1\\1\end{pmatrix}\right\}$

**Injective?** No (nullity ≠ 0). **Surjective?** No (rank = 2 < 3).

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Rank-Nullity Theorem | $\dim(V) = \text{nullity}(T) + \text{rank}(T)$ |
| For matrices | rank + nullity = number of columns |
| rank($A$) | Number of pivot columns |
| nullity($A$) | Number of free variables |
| Same-dim spaces | Injective ⟺ Surjective ⟺ Bijective |
| More vars than eqns | Homogeneous system always has non-trivial solution |
| Row rank = Column rank | $\text{rank}(A) = \text{rank}(A^T)$ |

---

## Quick Revision Questions

1. State the Rank-Nullity Theorem. What conditions are required on $V$?

2. If $T : \mathbb{R}^7 \to \mathbb{R}^4$ has rank 3, what is its nullity?

3. Can a linear map $T : \mathbb{R}^3 \to \mathbb{R}^5$ be surjective? Explain using Rank-Nullity.

4. Prove: if $A$ is $m \times n$ with $n > m$, then $A\mathbf{x} = \mathbf{0}$ has a non-trivial solution.

5. If $\dim(\ker(T)) = 0$ and $\dim(V) = \dim(W)$, what type of map is $T$?

6. For the matrix $A = \begin{pmatrix}1&2\\3&6\end{pmatrix}$, find rank, nullity, and verify the theorem.

---

[← Previous: Kernel and Range](02-kernel-and-range.md) · [Back to README](../README.md) · [Next: Matrix Representation →](04-matrix-representation.md)
