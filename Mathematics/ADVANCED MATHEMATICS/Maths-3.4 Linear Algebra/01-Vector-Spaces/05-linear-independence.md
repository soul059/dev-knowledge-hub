# 1.5 Linear Independence

[← Previous: Span of Vectors](04-span-of-vectors.md) · [Back to README](../README.md) · [Next: Basis and Dimension →](06-basis-and-dimension.md)

---

## Chapter Overview

**Linear independence** captures the idea that no vector in a set is "redundant" — none can be written as a combination of the others. This is the key concept that separates a minimal spanning set (a basis) from a bloated one. This chapter defines linear independence and dependence, gives multiple equivalent characterisations, and shows systematic methods to test it.

---

## 1. Definition

**Definition.** A set of vectors $\{\mathbf{v}_1, \mathbf{v}_2, \dots, \mathbf{v}_k\}$ in a vector space $V$ is called **linearly independent** if the only solution to

$$c_1 \mathbf{v}_1 + c_2 \mathbf{v}_2 + \cdots + c_k \mathbf{v}_k = \mathbf{0}$$

is the **trivial solution** $c_1 = c_2 = \cdots = c_k = 0$.

The set is **linearly dependent** if there exists a **non-trivial** solution (i.e., at least one $c_i \neq 0$).

---

## 2. Geometric Intuition

```
  LINEARLY INDEPENDENT (in ℝ²)          LINEARLY DEPENDENT (in ℝ²)

        y                                      y
        ▲                                      ▲
        │  v₂                                  │    v₂ = 2·v₁
        │ ╱                                    │   ╱
        │╱                                     │  ╱
   ─────┼──────▶ x                        ─────┼──╱───▶ x
        │       v₁                              │╱      v₁
        │                                       │
  v₁, v₂ point in                         v₁, v₂ point in the
  DIFFERENT directions                     SAME direction (one is
                                           a scalar multiple)
```

| Geometric Situation | Independent? |
|-|-|
| Two vectors in different directions | ✓ Independent |
| Two vectors on the same line | ✗ Dependent |
| Three vectors spanning a plane only | ✗ Dependent |
| Three vectors spanning all of $\mathbb{R}^3$ | ✓ Independent |

---

## 3. Equivalent Characterisations

The following are **equivalent** for a set $S = \{\mathbf{v}_1, \dots, \mathbf{v}_k\}$:

| # | Statement |
|---|-----------|
| 1 | $S$ is linearly dependent |
| 2 | At least one vector in $S$ is a linear combination of the others |
| 3 | The homogeneous system $c_1\mathbf{v}_1 + \cdots + c_k\mathbf{v}_k = \mathbf{0}$ has a non-trivial solution |
| 4 | Removing some vector from $S$ does not change $\text{span}(S)$ |
| 5 | (In $\mathbb{R}^n$) The matrix $A = [\mathbf{v}_1 \mid \cdots \mid \mathbf{v}_k]$ has a non-trivial null space |

**Proof of 1 ⟺ 2:**  
If $c_j \neq 0$ in a non-trivial relation, then $\mathbf{v}_j = -\frac{1}{c_j}\sum_{i \neq j} c_i \mathbf{v}_i$.  
Conversely, if $\mathbf{v}_j = \sum_{i \neq j} a_i \mathbf{v}_i$, then $\sum_{i \neq j} a_i \mathbf{v}_i + (-1)\mathbf{v}_j = \mathbf{0}$ is a non-trivial relation.

---

## 4. Testing Linear Independence (Systematic Method)

Given vectors in $\mathbb{R}^n$, form the matrix $A$ with them as **columns** and row-reduce:

$$A = [\mathbf{v}_1 \mid \mathbf{v}_2 \mid \cdots \mid \mathbf{v}_k]$$

```
  ┌──────────────────────────────────────┐
  │ Form matrix A = [v₁ | v₂ | … | vₖ]  │
  └─────────────────┬────────────────────┘
                    │
                    ▼
  ┌──────────────────────────────────────┐
  │ Row-reduce A to echelon form         │
  └─────────────────┬────────────────────┘
                    │
          ┌─────────┴──────────┐
          │ Every column has   │
          │ a pivot?           │
          ▼                    ▼
     ┌─────────┐         ┌───────────┐
     │   YES   │         │    NO     │
     │ Linearly│         │ Linearly  │
     │ INDEP.  │         │ DEPENDENT │
     └─────────┘         └───────────┘
```

- **Pivot in every column** → only trivial solution → **independent**
- **Free variable exists** → non-trivial solution → **dependent**

---

## 5. Worked Examples

### Example 1: Independent Set

Are $\mathbf{v}_1 = (1, 0, 0)$, $\mathbf{v}_2 = (0, 1, 0)$, $\mathbf{v}_3 = (0, 0, 1)$ independent?

```
  A = ┌         ┐       Already in echelon form!
      │ 1  0  0 │       Every column has a pivot.
      │ 0  1  0 │
      │ 0  0  1 │       ──▶  INDEPENDENT ✓
      └         ┘
```

### Example 2: Dependent Set

Are $\mathbf{v}_1 = (1, 2, 3)$, $\mathbf{v}_2 = (4, 5, 6)$, $\mathbf{v}_3 = (7, 8, 9)$ independent?

```
  ┌            ┐         ┌            ┐        ┌            ┐
  │ 1  4  7    │   R₂-2R₁│ 1  4   7   │  R₃-R₂ │ 1  4   7   │
  │ 2  5  8    │  ──────▶│ 0 -3  -6   │ ──────▶│ 0 -3  -6   │
  │ 3  6  9    │         │ 0 -6 -12   │        │ 0  0   0   │
  └            ┘         └            ┘        └            ┘
```

Column 3 has no pivot → free variable → **DEPENDENT**.

The dependence relation: from the echelon form, $c_3$ is free. Setting $c_3 = 1$: $c_2 = -2$, $c_1 = 1$.

$$1 \cdot (1,2,3) - 2 \cdot (4,5,6) + 1 \cdot (7,8,9) = (0,0,0) \quad ✓$$

### Example 3: Using Determinant (Square Case)

For $n$ vectors in $\mathbb{R}^n$, form the $n \times n$ matrix $A$:

$$\text{Independent} \iff \det(A) \neq 0$$

$$A = \begin{pmatrix} 1 & 2 \\ 3 & 4 \end{pmatrix}, \quad \det(A) = 4 - 6 = -2 \neq 0 \implies \text{Independent}$$

---

## 6. Important Theorems

### Theorem 1: More Vectors Than Dimension ⟹ Dependent

> If $V$ is a vector space of dimension $n$ (see Chapter 1.6), any set of more than $n$ vectors in $V$ is linearly dependent.

**Consequence:** In $\mathbb{R}^3$, any set of 4 or more vectors is dependent.

### Theorem 2: A Set Containing the Zero Vector Is Dependent

> If $\mathbf{0} \in \{\mathbf{v}_1, \dots, \mathbf{v}_k\}$, the set is linearly dependent.

**Proof:** Suppose $\mathbf{v}_1 = \mathbf{0}$. Then $1 \cdot \mathbf{0} + 0 \cdot \mathbf{v}_2 + \cdots + 0 \cdot \mathbf{v}_k = \mathbf{0}$ is a non-trivial relation.

### Theorem 3: Subset of Independent Set Is Independent

> If $\{\mathbf{v}_1, \dots, \mathbf{v}_k\}$ is independent, so is any non-empty subset.

### Theorem 4: Superset of Dependent Set Is Dependent

> If $\{\mathbf{v}_1, \dots, \mathbf{v}_k\}$ is dependent, so is any set containing it.

---

## 7. Decision Table

| Situation | Conclusion |
|-----------|------------|
| One vector $\mathbf{v}$ | Independent ⟺ $\mathbf{v} \neq \mathbf{0}$ |
| Two vectors $\mathbf{u}, \mathbf{v}$ | Independent ⟺ neither is a scalar multiple of the other |
| $k > n$ vectors in $\mathbb{R}^n$ | Always dependent |
| Set contains $\mathbf{0}$ | Always dependent |
| $n$ vectors in $\mathbb{R}^n$ | Independent ⟺ $\det \neq 0$ |
| Rows of $A$ | Independent ⟺ $\text{rank}(A) = m$ (number of rows) |

---

## 8. Wronskian — Testing Independence of Functions

For functions $f_1, f_2, \dots, f_k$ on an interval $I$, the **Wronskian** is:

$$W(f_1, \dots, f_k)(x) = \det \begin{pmatrix} f_1 & f_2 & \cdots & f_k \\ f_1' & f_2' & \cdots & f_k' \\ \vdots & \vdots & \ddots & \vdots \\ f_1^{(k-1)} & f_2^{(k-1)} & \cdots & f_k^{(k-1)} \end{pmatrix}$$

> If $W \neq 0$ at some point $x_0$, then $\{f_1, \dots, f_k\}$ are linearly independent on $I$.

**Example:** $f_1 = e^x$, $f_2 = e^{2x}$:

$$W = \det \begin{pmatrix} e^x & e^{2x} \\ e^x & 2e^{2x} \end{pmatrix} = 2e^{3x} - e^{3x} = e^{3x} \neq 0$$

So $e^x$ and $e^{2x}$ are linearly independent.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Linearly independent | Only trivial solution to $\sum c_i \mathbf{v}_i = \mathbf{0}$ |
| Linearly dependent | There exists a non-trivial relation |
| Testing (matrix method) | Row-reduce; pivot in every column ⟹ independent |
| Testing (determinant) | For $n$ vectors in $\mathbb{R}^n$: $\det \neq 0$ ⟹ independent |
| $k > n$ vectors in $\mathbb{R}^n$ | Always dependent |
| Zero vector rule | Set containing $\mathbf{0}$ is always dependent |
| Wronskian | Determinant test for function independence |

---

## Quick Revision Questions

1. Determine whether $\{(1, 2, 3), (4, 5, 6), (2, 1, 0)\}$ is linearly independent in $\mathbb{R}^3$.

2. Prove that any set containing the zero vector is linearly dependent.

3. Can a set of 4 vectors in $\mathbb{R}^3$ be linearly independent? Why or why not?

4. If $\{\mathbf{v}_1, \mathbf{v}_2\}$ is independent and $\mathbf{v}_3 \notin \text{span}\{\mathbf{v}_1, \mathbf{v}_2\}$, prove that $\{\mathbf{v}_1, \mathbf{v}_2, \mathbf{v}_3\}$ is independent.

5. Use the Wronskian to show that $\sin x$ and $\cos x$ are linearly independent in $C(\mathbb{R})$.

6. True or False: If the columns of a $3 \times 3$ matrix are linearly dependent, then $\det(A) = 0$.

---

[← Previous: Span of Vectors](04-span-of-vectors.md) · [Back to README](../README.md) · [Next: Basis and Dimension →](06-basis-and-dimension.md)
