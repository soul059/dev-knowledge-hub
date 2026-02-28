# 3.1 Inner Product Definition

[← Previous: Change of Basis](../02-Linear-Transformations/05-change-of-basis.md) · [Back to README](../README.md) · [Next: Norm and Distance →](02-norm-and-distance.md)

---

## Chapter Overview

Up to now, vector spaces have only algebraic structure (addition and scalar multiplication). An **inner product** adds *geometric* structure — it lets us talk about **lengths**, **angles**, and **perpendicularity**. This chapter defines inner products axiomatically and gives key examples.

---

## 1. Motivation

In $\mathbb{R}^2$ we know the dot product: $\mathbf{u} \cdot \mathbf{v} = u_1 v_1 + u_2 v_2$.

From it we derive:
- Length: $\|\mathbf{u}\| = \sqrt{\mathbf{u} \cdot \mathbf{u}}$
- Angle: $\cos\theta = \frac{\mathbf{u} \cdot \mathbf{v}}{\|\mathbf{u}\|\|\mathbf{v}\|}$
- Perpendicularity: $\mathbf{u} \perp \mathbf{v} \iff \mathbf{u} \cdot \mathbf{v} = 0$

An inner product **generalises** the dot product to arbitrary vector spaces.

---

## 2. Formal Definition

**Definition.** An **inner product** on a real vector space $V$ is a function $\langle \cdot, \cdot \rangle : V \times V \to \mathbb{R}$ satisfying for all $\mathbf{u}, \mathbf{v}, \mathbf{w} \in V$ and $c \in \mathbb{R}$:

| # | Axiom | Name |
|---|-------|------|
| IP1 | $\langle \mathbf{u}, \mathbf{v} \rangle = \langle \mathbf{v}, \mathbf{u} \rangle$ | Symmetry |
| IP2 | $\langle \mathbf{u} + \mathbf{v}, \mathbf{w} \rangle = \langle \mathbf{u}, \mathbf{w} \rangle + \langle \mathbf{v}, \mathbf{w} \rangle$ | Additivity (1st argument) |
| IP3 | $\langle c\mathbf{u}, \mathbf{v} \rangle = c\langle \mathbf{u}, \mathbf{v} \rangle$ | Homogeneity (1st argument) |
| IP4 | $\langle \mathbf{u}, \mathbf{u} \rangle \geq 0$, with equality iff $\mathbf{u} = \mathbf{0}$ | Positive-definiteness |

A vector space equipped with an inner product is called an **inner product space**.

> **Note (complex case):** For complex vector spaces, IP1 becomes $\langle \mathbf{u}, \mathbf{v} \rangle = \overline{\langle \mathbf{v}, \mathbf{u} \rangle}$ (conjugate symmetry), and IP3 uses $\langle c\mathbf{u}, \mathbf{v} \rangle = c\langle \mathbf{u}, \mathbf{v} \rangle$ (linearity in the first argument, conjugate-linearity in the second).

---

## 3. Bilinearity

From IP1–IP3, the inner product is also linear in the **second** argument:

$$\langle \mathbf{u}, c\mathbf{v} + d\mathbf{w} \rangle = c\langle \mathbf{u}, \mathbf{v} \rangle + d\langle \mathbf{u}, \mathbf{w} \rangle$$

So an inner product is a **bilinear, symmetric, positive-definite** form.

---

## 4. Examples

### Example 1: The Standard (Dot) Inner Product on $\mathbb{R}^n$

$$\langle \mathbf{u}, \mathbf{v} \rangle = \mathbf{u} \cdot \mathbf{v} = \sum_{i=1}^n u_i v_i = \mathbf{u}^T \mathbf{v}$$

This is the "default" inner product. When no inner product is specified, this is assumed.

### Example 2: Weighted Inner Product on $\mathbb{R}^n$

Fix positive weights $w_1, w_2, \dots, w_n > 0$:

$$\langle \mathbf{u}, \mathbf{v} \rangle_w = \sum_{i=1}^n w_i u_i v_i$$

**Verification:** IP1–IP3 follow from real arithmetic. IP4: $\sum w_i u_i^2 \geq 0$ (since each $w_i > 0$), and $= 0$ iff all $u_i = 0$.

### Example 3: Inner Product on $M_{m \times n}(\mathbb{R})$ (Frobenius)

$$\langle A, B \rangle = \text{tr}(A^T B) = \sum_{i,j} a_{ij} b_{ij}$$

This is the entry-by-entry dot product of two matrices.

### Example 4: Inner Product on $C[a,b]$

$$\langle f, g \rangle = \int_a^b f(x) g(x)\, dx$$

**Verification:**
- IP1: $\int f g = \int g f$ ✓
- IP2, IP3: Linearity of integration ✓
- IP4: $\int_a^b f(x)^2 dx \geq 0$, and $= 0$ iff $f = 0$ (since $f$ is continuous) ✓

### Example 5: Inner Product on $P_n$

$$\langle p, q \rangle = \int_0^1 p(x) q(x)\, dx$$

Or alternatively: $\langle p, q \rangle = \sum_{i=0}^{n} p(x_i) q(x_i)$ for distinct points $x_0, \dots, x_n$ (evaluation inner product).

---

## 5. Non-Examples

| Function | Why It Fails |
|----------|-------------|
| $\langle \mathbf{u}, \mathbf{v} \rangle = u_1 v_1 - u_2 v_2$ | Not positive-definite: $\langle (0,1), (0,1) \rangle = -1 < 0$ |
| $\langle \mathbf{u}, \mathbf{v} \rangle = (u_1 + v_1)^2$ | Not bilinear |
| $\langle \mathbf{u}, \mathbf{v} \rangle = 0$ for all $\mathbf{u}, \mathbf{v}$ | Not positive-definite (zero is the zero form) |

---

## 6. General Inner Product via Positive-Definite Matrix

Any inner product on $\mathbb{R}^n$ can be expressed as:

$$\langle \mathbf{u}, \mathbf{v} \rangle = \mathbf{u}^T A \mathbf{v}$$

where $A$ is a **symmetric positive-definite** (SPD) matrix.

| Condition on $A$ | Inner Product Axiom |
|-----------------|-------------------|
| $A = A^T$ | IP1 (symmetry) |
| $\mathbf{x}^T A \mathbf{x} > 0$ for $\mathbf{x} \neq 0$ | IP4 (positive-definiteness) |

**Example:** $A = \begin{pmatrix}2 & 1 \\ 1 & 3\end{pmatrix}$

$\langle (u_1,u_2), (v_1,v_2) \rangle = 2u_1 v_1 + u_1 v_2 + u_2 v_1 + 3u_2 v_2$

Check SPD: eigenvalues of $A$ are $\frac{5 \pm \sqrt{5}}{2}$, both positive ✓.

```
  Standard Dot Product       Weighted Inner Product
  (A = I)                    (A = SPD matrix)
  
  Unit "circle": x²+y²=1    Unit "circle": xᵀAx = 1
  
        y                          y
        ▲                          ▲
       ╱│╲                        ╱│╲
      / │ \                     ╱  │  ╲    (an ellipse!)
     /  │  \                  ╱    │    ╲
    (   │   )               (      │      )
     \  │  /                  ╲    │    ╱
      \ │ /                     ╲  │  ╱
       ╲│╱                        ╲│╱
   ─────┼──────▶ x          ──────┼──────▶ x
```

---

## 7. Inner Product Space Hierarchy

```
  ┌────────────────────────────────────┐
  │          Vector Space              │
  │      (addition, scalar mult.)      │
  │                                    │
  │   ┌────────────────────────────┐   │
  │   │    Normed Space            │   │
  │   │    (has a norm ‖·‖)        │   │
  │   │                            │   │
  │   │   ┌────────────────────┐   │   │
  │   │   │  Inner Product     │   │   │
  │   │   │  Space             │   │   │
  │   │   │  (⟨·,·⟩ defines   │   │   │
  │   │   │   norm and angle)  │   │   │
  │   │   └────────────────────┘   │   │
  │   └────────────────────────────┘   │
  └────────────────────────────────────┘
  
  Inner product ──▶ Norm ──▶ Metric ──▶ Topology
  (most structure)                    (least structure)
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Inner product | Bilinear, symmetric, positive-definite function $V \times V \to \mathbb{R}$ |
| 4 axioms | Symmetry, additivity, homogeneity, positive-definiteness |
| Standard inner product | $\langle \mathbf{u}, \mathbf{v} \rangle = \sum u_i v_i$ |
| Function space IP | $\langle f, g \rangle = \int f(x)g(x) dx$ |
| Matrix form | $\langle \mathbf{u}, \mathbf{v} \rangle = \mathbf{u}^T A \mathbf{v}$ ($A$ symmetric positive-definite) |
| Frobenius IP | $\langle A, B \rangle = \text{tr}(A^T B)$ |
| Inner product space | A vector space equipped with an inner product |

---

## Quick Revision Questions

1. State the four axioms of an inner product. Which axiom distinguishes a genuine inner product from a general bilinear form?

2. Verify that $\langle f, g \rangle = \int_0^1 f(x)g(x)\,dx$ satisfies all four inner product axioms on $C[0,1]$.

3. Does $\langle \mathbf{u}, \mathbf{v} \rangle = u_1 v_1 - u_2 v_2$ define an inner product on $\mathbb{R}^2$? Why or why not?

4. Show that $\langle \mathbf{u}, \mathbf{v} \rangle = \mathbf{u}^T A \mathbf{v}$ with $A = \begin{pmatrix}1&0\\0&4\end{pmatrix}$ is an inner product.

5. What is $\langle \sin x, \cos x \rangle$ using the inner product $\int_{-\pi}^{\pi} f(x)g(x)\,dx$?

6. Why must the matrix $A$ in $\langle \mathbf{u}, \mathbf{v} \rangle = \mathbf{u}^T A \mathbf{v}$ be symmetric for IP1 to hold?

---

[← Previous: Change of Basis](../02-Linear-Transformations/05-change-of-basis.md) · [Back to README](../README.md) · [Next: Norm and Distance →](02-norm-and-distance.md)
