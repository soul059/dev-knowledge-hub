# 1.1 Definition and Examples of Vector Spaces

[← Back to README](../README.md) · [Next: Subspaces →](02-subspaces.md)

---

## Chapter Overview

A **vector space** is the most fundamental object in Linear Algebra. Before studying transformations, eigenvalues, or decompositions, we must understand *what kind of objects* we are working with. This chapter introduces the formal definition of a vector space, walks through the axioms, and gives a rich set of examples (and non-examples).

---

## 1. Intuitive Motivation

In school we think of a "vector" as an arrow with magnitude and direction. That picture is useful but limited. Linear Algebra generalises the idea:

> **A vector is any object that can be added to other objects of its kind and multiplied by scalars, subject to a set of natural rules.**

Polynomials, matrices, functions, and sequences can all be "vectors" in the right context.

---

## 2. Formal Definition

**Definition.** A **vector space** over a field $F$ is a non-empty set $V$ together with two operations:

1. **Vector addition:** $+: V \times V \to V$
2. **Scalar multiplication:** $\cdot : F \times V \to V$

satisfying the following **10 axioms** for all $\mathbf{u}, \mathbf{v}, \mathbf{w} \in V$ and all $a, b \in F$:

### Axioms of a Vector Space

| # | Axiom Name | Statement |
|---|-----------|-----------|
| A1 | Closure under addition | $\mathbf{u} + \mathbf{v} \in V$ |
| A2 | Commutativity of addition | $\mathbf{u} + \mathbf{v} = \mathbf{v} + \mathbf{u}$ |
| A3 | Associativity of addition | $(\mathbf{u} + \mathbf{v}) + \mathbf{w} = \mathbf{u} + (\mathbf{v} + \mathbf{w})$ |
| A4 | Existence of additive identity | $\exists\; \mathbf{0} \in V : \mathbf{u} + \mathbf{0} = \mathbf{u}$ |
| A5 | Existence of additive inverse | $\forall\; \mathbf{u}, \exists\; (-\mathbf{u}) : \mathbf{u} + (-\mathbf{u}) = \mathbf{0}$ |
| S1 | Closure under scalar multiplication | $a \cdot \mathbf{u} \in V$ |
| S2 | Compatibility of scalar multiplication | $a(b\mathbf{u}) = (ab)\mathbf{u}$ |
| S3 | Identity element of scalar multiplication | $1 \cdot \mathbf{u} = \mathbf{u}$ |
| D1 | Distributivity over vector addition | $a(\mathbf{u} + \mathbf{v}) = a\mathbf{u} + a\mathbf{v}$ |
| D2 | Distributivity over scalar addition | $(a + b)\mathbf{u} = a\mathbf{u} + b\mathbf{u}$ |

> **Key insight:** The axioms do *not* define what vectors look like. They define how vectors *behave*. Any set + operations satisfying these rules is a vector space.

---

## 3. The Field of Scalars

The most common fields used:

| Field | Symbol | Elements |
|-------|--------|----------|
| Real numbers | $\mathbb{R}$ | All real numbers |
| Complex numbers | $\mathbb{C}$ | Numbers of the form $a + bi$ |
| Rational numbers | $\mathbb{Q}$ | All fractions $p/q$ |
| Finite field | $\mathbb{F}_p$ | Integers modulo a prime $p$ |

When we say "a vector space over $\mathbb{R}$", the scalars come from $\mathbb{R}$.

---

## 4. Examples of Vector Spaces

### Example 1: $\mathbb{R}^n$ — The Canonical Example

$\mathbb{R}^n = \{(x_1, x_2, \dots, x_n) \mid x_i \in \mathbb{R}\}$

Operations:
- $(x_1, \dots, x_n) + (y_1, \dots, y_n) = (x_1 + y_1, \dots, x_n + y_n)$
- $c(x_1, \dots, x_n) = (cx_1, \dots, cx_n)$

```
   Geometric picture of ℝ²         Geometric picture of ℝ³
                                    
        y                                z
        ▲                                ▲
        │   • (2,3)                      │   / y
        │  /                             │  /
        │ /                              │ /
        │/                               │/________▶ x
   ─────┼──────▶ x                      /│
        │                              /  │
        │                             ▼
```

**Verification sketch for ℝ²:**
- A1 ✓ : $(x_1+y_1, x_2+y_2) \in \mathbb{R}^2$
- A4 ✓ : $\mathbf{0} = (0,0)$
- A5 ✓ : $-\mathbf{u} = (-u_1, -u_2)$
- All other axioms follow from real-number arithmetic.

### Example 2: $M_{m \times n}(\mathbb{R})$ — The Space of Matrices

The set of all $m \times n$ matrices with real entries forms a vector space under usual matrix addition and scalar multiplication.

```
  ┌         ┐     ┌         ┐     ┌                 ┐
  │ a₁₁ a₁₂ │     │ b₁₁ b₁₂ │     │ a₁₁+b₁₁ a₁₂+b₁₂ │
  │ a₂₁ a₂₂ │  +  │ b₂₁ b₂₂ │  =  │ a₂₁+b₂₁ a₂₂+b₂₂ │
  └         ┘     └         ┘     └                 ┘
```

Zero vector = the zero matrix. Additive inverse = negate every entry.

### Example 3: $P_n(\mathbb{R})$ — Polynomials of Degree ≤ n

$P_n = \{a_0 + a_1 x + a_2 x^2 + \cdots + a_n x^n \mid a_i \in \mathbb{R}\}$

Operations: add coefficients, multiply each coefficient by the scalar.

### Example 4: $C[a,b]$ — Continuous Functions on $[a,b]$

The set of all continuous real-valued functions on $[a,b]$.

- $(f+g)(x) = f(x) + g(x)$
- $(cf)(x) = c \cdot f(x)$
- Zero vector = the zero function $f(x) = 0$ for all $x$.

### Example 5: The Trivial (Zero) Vector Space

$V = \{\mathbf{0}\}$ with $\mathbf{0} + \mathbf{0} = \mathbf{0}$ and $c \cdot \mathbf{0} = \mathbf{0}$.

All 10 axioms are satisfied. This is the **smallest possible** vector space.

---

## 5. Non-Examples (Common Pitfalls)

### Non-Example 1: $\mathbb{R}^+$ with Usual Addition and Scalar Multiplication

$V = \{x \in \mathbb{R} \mid x > 0\}$

**Fails** A4: there is no element $0 \in V$ (since $0 \notin \mathbb{R}^+$).  
**Fails** A5: the additive inverse of $3$ would be $-3$, but $-3 \notin V$.

### Non-Example 2: The Set $\{(x, y) \in \mathbb{R}^2 \mid x \geq 0\}$

This is the right half-plane. It is **not** closed under scalar multiplication:  
$(-1) \cdot (1, 0) = (-1, 0)$, which is not in the set.  
**Fails** S1.

### Non-Example 3: Custom Operations that Break Axioms

Let $V = \mathbb{R}$ with operations $x \oplus y = xy$ and $c \odot x = x^c$.

Check A4: Need $e$ such that $x \oplus e = xe = x$ for all $x$, so $e = 1$.  
Check A5: Need $y$ such that $xy = 1$, so $y = 1/x$. But $0$ has no inverse! Fails A5.

---

## 6. Diagram — Taxonomy of Vector Spaces

```
                    ┌──────────────────────────────┐
                    │      VECTOR SPACE (V, F)      │
                    │  10 axioms satisfied           │
                    └──────────────┬───────────────┘
                                   │
            ┌──────────────────────┼──────────────────────┐
            │                      │                      │
     ┌──────▼──────┐       ┌──────▼──────┐       ┌──────▼──────┐
     │   Finite-   │       │  Infinite-  │       │   Trivial   │
     │ Dimensional │       │ Dimensional │       │    {0}      │
     │  (ℝⁿ, Pₙ,  │       │  (C[a,b],   │       │             │
     │  Mₘₓₙ)     │       │   ℓ², …)    │       │             │
     └─────────────┘       └─────────────┘       └─────────────┘
```

---

## 7. Important Consequences of the Axioms

Once the 10 axioms are granted, several results follow *logically*:

| # | Property | Proof Sketch |
|---|----------|-------------|
| C1 | The zero vector $\mathbf{0}$ is unique | Assume two: $\mathbf{0}_1 = \mathbf{0}_1 + \mathbf{0}_2 = \mathbf{0}_2$ |
| C2 | The additive inverse of $\mathbf{v}$ is unique | If $\mathbf{v} + \mathbf{w}_1 = \mathbf{0}$ and $\mathbf{v} + \mathbf{w}_2 = \mathbf{0}$ then $\mathbf{w}_1 = \mathbf{w}_2$ |
| C3 | $0 \cdot \mathbf{v} = \mathbf{0}$ | $0\mathbf{v} = (0+0)\mathbf{v} = 0\mathbf{v} + 0\mathbf{v}$; cancel |
| C4 | $c \cdot \mathbf{0} = \mathbf{0}$ | $c\mathbf{0} = c(\mathbf{0}+\mathbf{0}) = c\mathbf{0} + c\mathbf{0}$; cancel |
| C5 | $(-1)\mathbf{v} = -\mathbf{v}$ | $\mathbf{v} + (-1)\mathbf{v} = 1\mathbf{v} + (-1)\mathbf{v} = (1-1)\mathbf{v} = 0\mathbf{v} = \mathbf{0}$ |
| C6 | If $c\mathbf{v} = \mathbf{0}$ then $c=0$ or $\mathbf{v}=\mathbf{0}$ | Assume $c \neq 0$; multiply both sides by $c^{-1}$ |

---

## 8. Real-World Applications

| Application | Vector Space Used |
|-------------|------------------|
| 3-D graphics / physics | $\mathbb{R}^3$ — positions, velocities, forces |
| Image processing | $M_{m \times n}(\mathbb{R})$ — a greyscale image is a matrix |
| Signal processing | Function spaces — signals are functions of time |
| Machine learning | $\mathbb{R}^d$ — feature vectors |
| Quantum mechanics | $\mathbb{C}^n$ — state vectors |
| Coding theory | $\mathbb{F}_2^n$ — binary codewords |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Vector space | A set $V$ + field $F$ + two operations satisfying 10 axioms |
| field $F$ | Supplies the scalars (commonly $\mathbb{R}$ or $\mathbb{C}$) |
| 10 axioms | 5 for addition (group), 5 for scalar multiplication + distributivity |
| Zero vector | Unique; satisfies $\mathbf{v} + \mathbf{0} = \mathbf{v}$ |
| Additive inverse | Unique; equals $(-1)\mathbf{v}$ |
| Standard examples | $\mathbb{R}^n$, $M_{m \times n}$, $P_n$, $C[a,b]$ |
| Verification strategy | Check all 10 axioms; failure of even one ⇒ not a vector space |

---

## Quick Revision Questions

1. **State the 10 axioms** of a vector space. Why is closure listed as an axiom?

2. **True or False:** The set of all $2 \times 2$ matrices with determinant 1 forms a vector space under standard matrix addition and scalar multiplication. Justify.

3. Show that for any vector $\mathbf{v}$ in a vector space, $0 \cdot \mathbf{v} = \mathbf{0}$.

4. Is the set $\{(x, y) \in \mathbb{R}^2 \mid x + y = 0\}$ a vector space under the usual operations of $\mathbb{R}^2$? Verify the necessary axioms.

5. Give an example of a set with addition and scalar multiplication defined, where *exactly one* axiom fails.

6. Why does the set of polynomials of degree *exactly* $n$ (not "at most $n$") fail to be a vector space?

---

[← Back to README](../README.md) · [Next: Subspaces →](02-subspaces.md)
