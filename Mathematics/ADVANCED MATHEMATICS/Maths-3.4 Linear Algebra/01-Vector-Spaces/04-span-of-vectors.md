# 1.4 Span of Vectors

[← Previous: Linear Combination](03-linear-combination.md) · [Back to README](../README.md) · [Next: Linear Independence →](05-linear-independence.md)

---

## Chapter Overview

The **span** of a set of vectors is the collection of *all possible* linear combinations of those vectors. It answers the question: "What is the reach of these vectors?" This chapter defines span, shows how to compute and visualise it, and proves that a span is always a subspace.

---

## 1. Definition

**Definition.** Let $S = \{\mathbf{v}_1, \mathbf{v}_2, \dots, \mathbf{v}_k\}$ be a set of vectors in a vector space $V$. The **span** of $S$ is:

$$\text{span}(S) = \text{span}\{\mathbf{v}_1, \dots, \mathbf{v}_k\} = \{c_1\mathbf{v}_1 + c_2\mathbf{v}_2 + \cdots + c_k\mathbf{v}_k \mid c_1, \dots, c_k \in F\}$$

In words: span($S$) is the set of **all linear combinations** of vectors in $S$.

**Convention:** $\text{span}(\varnothing) = \{\mathbf{0}\}$.

---

## 2. Geometric Interpretation

```
  In ℝ³:

  span{v₁} = a line                 span{v₁, v₂} = a plane
  through the origin                through the origin
  (if v₁ ≠ 0)                      (if v₁, v₂ not parallel)

        │    / v₁                         ╱ v₂
        │   /                           ╱
        │  /                     ──────╱────────── plane
        │ /                          ╱    v₁
        │/                         ╱───────▶
   ─────┼──────                  ╱
        │                       ╱

  span{v₁, v₂, v₃} = ℝ³         span{0} = {0}
  (if none is a combination      (a single point)
   of the other two)
```

| Number of Vectors (in ℝ³) | Geometric Object | Condition |
|---------------------------|------------------|-----------|
| 0 (or only **0**) | Point (origin) | — |
| 1 non-zero vector | Line through origin | $\mathbf{v}_1 \neq \mathbf{0}$ |
| 2 non-parallel vectors | Plane through origin | $\mathbf{v}_2 \neq c \mathbf{v}_1$ |
| 3 "independent" vectors | All of $\mathbb{R}^3$ | No vector is a combination of others |

---

## 3. Span Is Always a Subspace

> **Theorem.** For any set $S \subseteq V$, $\text{span}(S)$ is a **subspace** of $V$.

**Proof (using the subspace test):**

Let $W = \text{span}\{\mathbf{v}_1, \dots, \mathbf{v}_k\}$.

1. **$\mathbf{0} \in W$:** Take all coefficients = 0. Then $0\mathbf{v}_1 + \cdots + 0\mathbf{v}_k = \mathbf{0} \in W$. ✓

2. **Closed under addition:** If $\mathbf{u} = \sum a_i \mathbf{v}_i$ and $\mathbf{w} = \sum b_i \mathbf{v}_i$, then:
$$\mathbf{u} + \mathbf{w} = \sum (a_i + b_i)\mathbf{v}_i \in W$$ ✓

3. **Closed under scalar multiplication:** $c\mathbf{u} = \sum (ca_i)\mathbf{v}_i \in W$. ✓

Moreover, $\text{span}(S)$ is the **smallest** subspace of $V$ that contains $S$.

---

## 4. Determining Whether a Vector Lies in a Span

**Question:** Is $\mathbf{w} \in \text{span}\{\mathbf{v}_1, \dots, \mathbf{v}_k\}$?

This is the same as asking: "Is $\mathbf{w}$ a linear combination of $\mathbf{v}_1, \dots, \mathbf{v}_k$?"

### Algorithm

```
  ┌─────────────────────────────────────────────┐
  │  Form augmented matrix [v₁ | v₂ | … | vₖ | w] │
  └───────────────────────┬─────────────────────┘
                          │
                          ▼
  ┌─────────────────────────────────────────────┐
  │  Row-reduce to echelon form                 │
  └───────────────────────┬─────────────────────┘
                          │
               ┌──────────┴──────────┐
               │ Last column          │
               │ has a pivot?         │
               ▼                      ▼
          ┌────────┐            ┌───────────┐
          │  YES   │            │    NO     │
          │ Incon- │            │ Consistent │
          │ sistent│            │ w ∈ span  │
          │ w ∉ span│           └───────────┘
          └────────┘
```

---

## 5. Worked Examples

### Example 1

Does $(3, 5, 7) \in \text{span}\{(1, 1, 1), (1, 2, 3)\}$?

Solve $c_1(1,1,1) + c_2(1,2,3) = (3,5,7)$:

```
  ┌             ┐        ┌             ┐       ┌             ┐
  │ 1  1 │ 3    │   R₂-R₁│ 1  1 │ 3    │   R₃-R₂│ 1  1 │ 3    │
  │ 1  2 │ 5    │  ────▶ │ 0  1 │ 2    │  ────▶ │ 0  1 │ 2    │
  │ 1  3 │ 7    │        │ 0  2 │ 4    │        │ 0  0 │ 0    │
  └             ┘        └             ┘        └             ┘
```

Consistent! $c_2 = 2,\; c_1 = 1$. So $(3,5,7) = 1(1,1,1) + 2(1,2,3)$. ✓

### Example 2

What is $\text{span}\{(1, 0), (0, 1)\}$ in $\mathbb{R}^2$?

Any $(a, b) = a(1,0) + b(0,1)$. So $\text{span}\{(1,0),(0,1)\} = \mathbb{R}^2$.

### Example 3 — Redundant Vectors

$\text{span}\{(1,0), (0,1), (1,1)\}$ in $\mathbb{R}^2$?

$(1,1) = 1(1,0) + 1(0,1)$ — redundant! The span is still $\mathbb{R}^2$.

> **Key idea:** Adding a vector that is already in the span does **not** enlarge the span.

---

## 6. Spanning Sets

**Definition.** A set $S$ is said to **span** (or be a **spanning set** for) $V$ if $\text{span}(S) = V$.

This means every vector in $V$ can be written as a linear combination of vectors in $S$.

### Example

$\{(1,0,0), (0,1,0), (0,0,1)\}$ spans $\mathbb{R}^3$.

$\{(1,0,0), (0,1,0)\}$ does **not** span $\mathbb{R}^3$ — e.g., $(0,0,1) \notin \text{span}$.

---

## 7. Properties of Span

| Property | Statement |
|----------|-----------|
| Subset | $S \subseteq \text{span}(S)$ |
| Monotonicity | $S_1 \subseteq S_2 \Rightarrow \text{span}(S_1) \subseteq \text{span}(S_2)$ |
| Idempotence | $\text{span}(\text{span}(S)) = \text{span}(S)$ |
| Subspace | $\text{span}(S)$ is the smallest subspace containing $S$ |
| Redundancy | If $\mathbf{v} \in \text{span}(S \setminus \{\mathbf{v}\})$ then $\text{span}(S) = \text{span}(S \setminus \{\mathbf{v}\})$ |

---

## 8. Column Space and Row Space

For a matrix $A \in M_{m \times n}(\mathbb{R})$:

| Space | Definition | Lives in |
|-------|-----------|----------|
| **Column space** $\text{Col}(A)$ | $\text{span}$ of the columns of $A$ | $\mathbb{R}^m$ |
| **Row space** $\text{Row}(A)$ | $\text{span}$ of the rows of $A$ | $\mathbb{R}^n$ |

$$\text{Col}(A) = \{A\mathbf{x} \mid \mathbf{x} \in \mathbb{R}^n\}$$

The column space answers: "What vectors $\mathbf{b}$ make $A\mathbf{x} = \mathbf{b}$ solvable?"

---

## 9. Real-World Application: RGB Colour Space

Every colour on a screen is a linear combination of **Red**, **Green**, **Blue**:

$$\text{colour} = r \cdot \text{R} + g \cdot \text{G} + b \cdot \text{B}, \quad r, g, b \in [0, 255]$$

The set $\{\text{R}, \text{G}, \text{B}\}$ spans the visible colour space on that display.

```
  Colour = r·R + g·G + b·B
  
  ┌──────┐   ┌──────┐   ┌──────┐
  │  R   │ + │  G   │ + │  B   │  =  displayed colour
  │(255, │   │(0,   │   │(0,   │
  │ 0,0) │   │255,0)│   │0,255)│
  └──────┘   └──────┘   └──────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| $\text{span}(S)$ | Set of all linear combinations of vectors in $S$ |
| Always a subspace | Contains $\mathbf{0}$, closed under $+$ and $\cdot$ |
| Smallest subspace | $\text{span}(S)$ is the smallest subspace containing $S$ |
| Spanning set for $V$ | $\text{span}(S) = V$ |
| Redundant vector | A vector already in the span of the rest — removing it doesn't change the span |
| Checking membership | Row-reduce the augmented matrix $[\mathbf{v}_1 \mid \cdots \mid \mathbf{v}_k \mid \mathbf{w}]$ |
| Column space | $\text{span}$ of columns = set of achievable $A\mathbf{x}$ |

---

## Quick Revision Questions

1. Find $\text{span}\{(1, 2), (2, 4)\}$ in $\mathbb{R}^2$. Describe it geometrically.

2. Is $(1, 1, 1) \in \text{span}\{(1, 0, 0), (0, 1, 0)\}$? Why or why not?

3. Prove that $\text{span}(S)$ is the smallest subspace of $V$ containing $S$.

4. Give a spanning set for the subspace $W = \{(a, b, a+b) \mid a, b \in \mathbb{R}\}$ of $\mathbb{R}^3$.

5. If $\text{span}\{v_1, v_2, v_3\} = \mathbb{R}^3$, and $v_3 = v_1 + v_2$, does $\text{span}\{v_1, v_2\} = \mathbb{R}^3$? Explain.

6. What is the column space of the identity matrix $I_n$?

---

[← Previous: Linear Combination](03-linear-combination.md) · [Back to README](../README.md) · [Next: Linear Independence →](05-linear-independence.md)
