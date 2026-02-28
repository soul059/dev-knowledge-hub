# 1.3 Linear Combination

[← Previous: Subspaces](02-subspaces.md) · [Back to README](../README.md) · [Next: Span of Vectors →](04-span-of-vectors.md)

---

## Chapter Overview

A **linear combination** is the most basic building-block operation in Linear Algebra. Every concept that follows — span, linear independence, basis, dimension — is defined in terms of linear combinations. This chapter defines the idea, provides geometric intuition, and shows how to determine whether a given vector is a linear combination of others.

---

## 1. Definition

**Definition.** Let $\mathbf{v}_1, \mathbf{v}_2, \dots, \mathbf{v}_k$ be vectors in a vector space $V$ over $F$, and let $c_1, c_2, \dots, c_k \in F$. The vector

$$\mathbf{w} = c_1 \mathbf{v}_1 + c_2 \mathbf{v}_2 + \cdots + c_k \mathbf{v}_k$$

is called a **linear combination** of $\mathbf{v}_1, \dots, \mathbf{v}_k$ with **coefficients** (or **weights**) $c_1, \dots, c_k$.

---

## 2. Geometric Interpretation in $\mathbb{R}^2$

```
        y
        ▲
        │         v₂ = (0,2)
        │         ▲
        │        /│
        │       / │
        │      /  │  w = 2v₁ + 1v₂ = (4,2)
        │     /   │        •
        │    /    │       ╱
        │   /     │      ╱
        │  /      │     ╱  2·v₁ = (4,0)
        │ /       │    ╱
   ─────┼─────────┼───╱─────────▶ x
        │         │
        │    v₁ = (2,0)
        │    ──────▶

  w = 2(2,0) + 1(0,2) = (4,0) + (0,2) = (4,2)
```

The idea: **scale** each vector, then **add** them tail-to-head.

---

## 3. How to Check: Is $\mathbf{w}$ a Linear Combination of $\mathbf{v}_1, \dots, \mathbf{v}_k$?

This question is equivalent to: **does the system of equations**

$$c_1 \mathbf{v}_1 + c_2 \mathbf{v}_2 + \cdots + c_k \mathbf{v}_k = \mathbf{w}$$

**have a solution** for $c_1, \dots, c_k$?

### Procedure

1. Set up the augmented matrix $[\mathbf{v}_1 \mid \mathbf{v}_2 \mid \cdots \mid \mathbf{v}_k \mid \mathbf{w}]$.
2. Row-reduce to echelon form.
3. If the system is **consistent** → $\mathbf{w}$ is a linear combination. Read off coefficients.
4. If the system is **inconsistent** → $\mathbf{w}$ is NOT a linear combination.

---

## 4. Worked Example

**Problem.** Is $\mathbf{w} = (1, 2, 3)$ a linear combination of $\mathbf{v}_1 = (1, 0, 1)$ and $\mathbf{v}_2 = (0, 1, 1)$?

We need to solve $c_1(1,0,1) + c_2(0,1,1) = (1,2,3)$:

$$\begin{cases} c_1 = 1 \\ c_2 = 2 \\ c_1 + c_2 = 3 \end{cases}$$

Setting up the augmented matrix and row-reducing:

```
  ┌             ┐       ┌             ┐
  │ 1  0 │ 1    │       │ 1  0 │ 1    │
  │ 0  1 │ 2    │  ──▶  │ 0  1 │ 2    │
  │ 1  1 │ 3    │       │ 0  0 │ 0    │   (R₃ ← R₃ - R₁)
  └             ┘       └             ┘
```

**Consistent!** $c_1 = 1,\; c_2 = 2$.

$$\mathbf{w} = 1 \cdot \mathbf{v}_1 + 2 \cdot \mathbf{v}_2 = (1,0,1) + (0,2,2) = (1,2,3)\; ✓$$

---

## 5. Another Example — Inconsistent Case

**Problem.** Is $\mathbf{w} = (1, 0, 0)$ a linear combination of $\mathbf{v}_1 = (1, 1, 0)$ and $\mathbf{v}_2 = (0, 1, 1)$?

$$\begin{cases} c_1 = 1 \\ c_1 + c_2 = 0 \\ c_2 = 0 \end{cases}$$

From equation 1: $c_1 = 1$. From equation 3: $c_2 = 0$. But equation 2: $1 + 0 = 1 \neq 0$. **Inconsistent!**

```
  ┌             ┐       ┌             ┐
  │ 1  0 │ 1    │       │ 1  0 │  1   │
  │ 1  1 │ 0    │  ──▶  │ 0  1 │ -1   │
  │ 0  1 │ 0    │       │ 0  0 │  1   │  ◄── 0 = 1  CONTRADICTION
  └             ┘       └             ┘
```

$\mathbf{w}$ is **not** a linear combination of $\mathbf{v}_1, \mathbf{v}_2$.

---

## 6. Linear Combinations in Different Vector Spaces

### Polynomials

Is $p(x) = 3 + 5x + x^2$ a linear combination of $p_1(x) = 1 + x$ and $p_2(x) = 1 + x^2$?

$c_1(1 + x) + c_2(1 + x^2) = 3 + 5x + x^2$

$(c_1 + c_2) + c_1 x + c_2 x^2 = 3 + 5x + x^2$

$$\begin{cases} c_1 + c_2 = 3 \\ c_1 = 5 \\ c_2 = 1 \end{cases}$$

Check: $c_1 + c_2 = 5 + 1 = 6 \neq 3$. **Not** a linear combination.

### Matrices

Is $B = \begin{pmatrix} 1 & 0 \\ 0 & 1 \end{pmatrix}$ a linear combination of $A_1 = \begin{pmatrix} 1 & 1 \\ 0 & 0 \end{pmatrix}$, $A_2 = \begin{pmatrix} 0 & 0 \\ 1 & 1 \end{pmatrix}$?

$c_1 A_1 + c_2 A_2 = B$ gives:

$$\begin{cases} c_1 = 1 \\ c_1 = 0 \end{cases} \quad \text{Contradiction. Not a linear combination.}$$

---

## 7. Connection to Systems of Linear Equations

The equation $A\mathbf{x} = \mathbf{b}$ asks:

> "Is $\mathbf{b}$ a linear combination of the **columns** of $A$?"

If $A = [\mathbf{a}_1 \mid \mathbf{a}_2 \mid \cdots \mid \mathbf{a}_n]$, then:

$$A\mathbf{x} = x_1 \mathbf{a}_1 + x_2 \mathbf{a}_2 + \cdots + x_n \mathbf{a}_n$$

```
  ┌               ┐ ┌     ┐     ┌     ┐
  │ a₁₁ a₁₂ a₁₃  │ │ x₁  │     │ b₁  │
  │ a₂₁ a₂₂ a₂₃  │ │ x₂  │  =  │ b₂  │
  │ a₃₁ a₃₂ a₃₃  │ │ x₃  │     │ b₃  │
  └               ┘ └     ┘     └     ┘

  ≡  x₁·col₁ + x₂·col₂ + x₃·col₃ = b
```

This "column picture" of $A\mathbf{x} = \mathbf{b}$ is one of the most important viewpoints in Linear Algebra.

---

## 8. Trivial and Non-Trivial Linear Combinations

| Type | Condition | Example |
|------|-----------|---------|
| **Trivial** | All coefficients are zero: $c_1 = c_2 = \cdots = c_k = 0$ | $0\mathbf{v}_1 + 0\mathbf{v}_2 = \mathbf{0}$ |
| **Non-trivial** | At least one coefficient is non-zero | $2\mathbf{v}_1 + (-3)\mathbf{v}_2 = \mathbf{0}$ |

This distinction becomes crucial when we define **linear independence** in Chapter 1.5.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Linear combination | $c_1\mathbf{v}_1 + \cdots + c_k\mathbf{v}_k$ |
| Coefficients | Scalars $c_1, \dots, c_k$ from the field $F$ |
| Checking method | Set up augmented matrix, row-reduce, check consistency |
| Column picture | $A\mathbf{x} = \mathbf{b}$ ⟺ Is $\mathbf{b}$ a linear combination of columns of $A$? |
| Trivial combination | All coefficients zero; always gives $\mathbf{0}$ |
| Non-trivial combination | At least one non-zero coefficient |

---

## Quick Revision Questions

1. Express $(7, 1, -1)$ as a linear combination of $(1, 1, 0)$, $(0, 1, 1)$, and $(1, 0, 1)$, or show it is impossible.

2. Is the polynomial $3x^2 + 2x + 1$ a linear combination of $x^2 + 1$ and $x^2 + x$? Justify.

3. Explain why the equation $A\mathbf{x} = \mathbf{b}$ can be interpreted as a linear combination problem.

4. True or False: The zero vector is always a linear combination of any set of vectors.

5. Can a vector be expressed as a linear combination of a single vector? Under what condition?

6. If $\mathbf{w} = 3\mathbf{v}_1 - 2\mathbf{v}_2 + 0\mathbf{v}_3$, is this a trivial or non-trivial linear combination?

---

[← Previous: Subspaces](02-subspaces.md) · [Back to README](../README.md) · [Next: Span of Vectors →](04-span-of-vectors.md)
