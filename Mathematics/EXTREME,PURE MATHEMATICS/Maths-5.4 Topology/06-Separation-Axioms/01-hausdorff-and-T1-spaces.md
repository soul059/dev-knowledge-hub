# Chapter 6.1 — Hausdorff, T₁, and T₀ Spaces

[← Previous: One-Point Compactification](../05-Compactness/05-one-point-compactification.md) | [Back to README](../README.md) | [Next: Regular and Normal Spaces →](02-regular-and-normal-spaces.md)

---

## 1. Overview

The **separation axioms** form a hierarchy of increasingly strong conditions on how well points and closed sets can be "separated" by open sets. These axioms ensure that topological spaces have enough open sets for analysis, geometry, and applications.

---

## 2. The Axioms

### 2.1 T₀ (Kolmogorov)

> **Def 1.1.** $X$ is **T₀** if for any two distinct points, there is an open set containing one but not the other.

$$\forall\, x \ne y, \;\exists\, U \text{ open}: (x \in U, y \notin U) \;\text{or}\; (y \in U, x \notin U)$$

### 2.2 T₁ (Fréchet)

> **Def 1.2.** $X$ is **T₁** if for any two distinct points, each has an open neighbourhood not containing the other.

$$\forall\, x \ne y, \;\exists\, U,V \text{ open}: x \in U,\, y \notin U \;\text{and}\; y \in V,\, x \notin V$$

### 2.3 T₂ (Hausdorff)

> **Def 1.3.** $X$ is **T₂** (Hausdorff) if any two distinct points can be separated by *disjoint* open sets.

$$\forall\, x \ne y, \;\exists\, U, V \text{ open, disjoint}: x \in U,\, y \in V$$

---

## 3. ASCII Diagram — Separation Strength

```
T₀:                    T₁:                    T₂ (Hausdorff):
 ┌─────────┐           ┌─────────┐            ┌─────────┐
 │  ┌───┐  │           │ ┌──┐┌──┐│            │ ┌──┐┌──┐│
 │  │ x │  │           │ │x ││ y││            │ │x ││ y││
 │  │   y  │           │ └──┘└──┘│            │ └──┘└──┘│
 │  └───┘  │           │         │            │  ↑    ↑  │
 └─────────┘           └─────────┘            │  U    V  │
 U contains x          U∋x, V∋y              │  U∩V=∅  │
 but not y            (not disjoint            └─────────┘
                       necessarily)
```

---

## 4. Equivalent Characterisations

### 4.1 T₁ Equivalences

> **Thm 1.4.** The following are equivalent for a space $X$:
> 1. $X$ is T₁.
> 2. Every singleton $\{x\}$ is closed.
> 3. Every finite set is closed.
> 4. For each $x$, $\{x\} = \bigcap \{U : U \text{ open}, x \in U\}$.

*Proof (1⟹2):* Let $y \ne x$. By T₁, $\exists U_y$ open with $y \in U_y$, $x \notin U_y$. Then $X \setminus \{x\} = \bigcup_{y \ne x} U_y$ is open, so $\{x\}$ is closed.

*Proof (2⟹1):* Given $x \ne y$, $X \setminus \{y\}$ is open and contains $x$ but not $y$; symmetrically for the other direction. ∎

### 4.2 Hausdorff Equivalences

> **Thm 1.5.** The following are equivalent for $X$:
> 1. $X$ is Hausdorff.
> 2. The diagonal $\Delta = \{(x,x) : x \in X\}$ is closed in $X \times X$.
> 3. Limits of nets (and filters) are unique.

---

## 5. Uniqueness of Limits

> **Thm 1.6.** In a Hausdorff space, a net converges to **at most one** limit.

*Proof:* Suppose $(x_\alpha) \to p$ and $(x_\alpha) \to q$ with $p \ne q$. Let $U, V$ be disjoint open sets with $p \in U$, $q \in V$. Eventually $x_\alpha \in U$ and $x_\alpha \in V$. But $U \cap V = \emptyset$, contradiction. ∎

In a non-Hausdorff space (e.g., cofinite topology on infinite set), a sequence can converge to every point.

---

## 6. Important Examples

| Space | T₀ | T₁ | T₂ |
|-------|:--:|:--:|:--:|
| Discrete space | ✓ | ✓ | ✓ |
| $\mathbb{R}$ (standard) | ✓ | ✓ | ✓ |
| Indiscrete ($|X| > 1$) | ✗ | ✗ | ✗ |
| Cofinite (infinite $X$) | ✓ | ✓ | ✗ |
| Sierpiński $\{0,1\}$ | ✓ | ✗ | ✗ |
| $\text{Spec}(R)$ (Zariski) | ✓ | varies | rarely |
| Particular-point topology | ✓ | ✗ | ✗ |

---

## 7. Permanence Properties

| Property | Subspaces | Products | Quotients |
|----------|:-:|:-:|:-:|
| T₀ | ✓ | ✓ | ✗ |
| T₁ | ✓ | ✓ | ✗ |
| T₂ | ✓ | ✓ | ✗ |

**Quotient Warning:** Identifying two points in a Hausdorff space can destroy T₂. The "line with two origins" ($\mathbb{R}$ with 0 doubled) is T₁ but not T₂.

---

## 8. Hausdorff and Compactness

> **Thm 1.7.** Let $f : X \to Y$ be continuous, bijective. If $X$ is compact and $Y$ is Hausdorff, then $f$ is a homeomorphism.

*Proof:* Need $f$ closed. A closed subset $C \subseteq X$ is compact (closed in compact). Then $f(C)$ is compact (continuous image). Compact in Hausdorff is closed. ∎

> **Thm 1.8.** A compact Hausdorff space is **normal** (T₄).

---

## 9. The Separation Hierarchy — Complete Picture

```
              T₀  ←──  T₁  ←──  T₂  ←──  T₃  ←──  T₃½  ←──  T₄
          Kolmogorov  Fréchet  Hausdorff  Regular  Completely   Normal
                                                    Regular
                                                   (Tychonoff)

  Increasing strength ──────────────────────────────────────→
```

---

## 10. Summary Table

| Axiom | Requirement | Key Characterisation |
|-------|------------|---------------------|
| T₀ | Distinguish points by some open set | Kolmogorov quotient |
| T₁ | Singletons are closed | Finite sets are closed |
| T₂ | Disjoint open nbhds | Unique limits; diagonal closed |
| Hausdorff + compact | Bijective cont. maps are homeo. | Compact ⊂ Hausdorff = closed |

---

## 11. Quick Revision Questions

1. State the definitions of T₀, T₁, and T₂. Give an example that is T₁ but not T₂.

2. Prove: $X$ is T₁ iff every singleton is closed.

3. Prove: $X$ is Hausdorff iff the diagonal is closed in $X \times X$.

4. Show that limits of sequences are unique in Hausdorff spaces.

5. Is the quotient of a Hausdorff space necessarily Hausdorff? Give a counterexample.

6. Prove that a continuous bijection from a compact space to a Hausdorff space is a homeomorphism.

---

[← Previous: One-Point Compactification](../05-Compactness/05-one-point-compactification.md) | [Back to README](../README.md) | [Next: Regular and Normal Spaces →](02-regular-and-normal-spaces.md)
