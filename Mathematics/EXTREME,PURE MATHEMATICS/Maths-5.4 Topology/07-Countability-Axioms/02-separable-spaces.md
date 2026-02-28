# Chapter 7.2 — Separable Spaces

[← Previous: First and Second Countable](01-first-and-second-countable.md) | [Back to README](../README.md) | [Next: Lindelöf Spaces →](03-lindelof-spaces.md)

---

## 1. Overview

A **separable** space has a countable dense subset. This is the weakest of the classical countability conditions but has profound consequences: separability of Hilbert and Banach spaces underpins functional analysis, and in metric spaces, separability is equivalent to second countability.

---

## 2. Definition

> **Def 2.1.** $X$ is **separable** if there exists a countable set $D \subseteq X$ with $\overline{D} = X$.

That is, every nonempty open set meets $D$.

---

## 3. Examples

| Space | Separable? | Dense subset |
|-------|:---:|-----------|
| $\mathbb{R}$ | ✓ | $\mathbb{Q}$ |
| $\mathbb{R}^n$ | ✓ | $\mathbb{Q}^n$ |
| $\ell^2$ | ✓ | Sequences with finitely many rational entries |
| $C([0,1])$ (sup norm) | ✓ | Polynomials with rational coefficients (Weierstrass) |
| $L^p([0,1])$, $1 \leq p < \infty$ | ✓ | Step functions with rational values |
| $\ell^\infty$ | ✗ | Uncountably many $\{0,1\}$-sequences at distance 1 |
| Discrete uncountable | ✗ | Only $D = X$ is dense |
| Sorgenfrey line | ✓ | $\mathbb{Q}$ (every $[a,b)$ contains a rational) |
| $\mathbb{R}_\ell \times \mathbb{R}_\ell$ | ✗ | The anti-diagonal is uncountable discrete |

---

## 4. Separability vs Other Countability Axioms

```
In general:
  2nd countable ──→ Separable       (take one point from each basis element)
  2nd countable ──→ Lindelöf
  Separable    ──/→ 2nd countable   (Sorgenfrey line: separable, not 2nd ctbl)
  Lindelöf     ──/→ Separable       (ω₁ + 1 is compact hence Lindelöf, but not separable)
  
In METRIC spaces:
  2nd countable ⟺ Separable ⟺ Lindelöf
```

---

## 5. Key Theorems

### 5.1 Second Countable ⟹ Separable

> **Thm 2.2.** Every second-countable space is separable.

*Proof:* Let $\mathcal{B} = \{B_n\}$ be a countable basis. Pick $x_n \in B_n$ for each $n$. Then $D = \{x_n\}$ is countable and dense: for any open $U$, $U \supseteq B_n$ for some $n$, so $x_n \in U \cap D$. ∎

### 5.2 Separable Metric Space ⟹ Second Countable

> **Thm 2.3.** A separable metric space is second countable.

*Proof:* Let $D = \{d_1, d_2, \ldots\}$ be countable dense. Then $\mathcal{B} = \{B(d_n, 1/m) : n, m \in \mathbb{N}\}$ is a countable basis. For any $x \in U$ (open), pick $\epsilon > 0$ with $B(x, \epsilon) \subseteq U$, then pick $d_n \in B(x, \epsilon/2) \cap D$, then $x \in B(d_n, \epsilon/2) \subseteq B(x, \epsilon) \subseteq U$. ∎

### 5.3 Separability and Products

> **Thm 2.4.** A countable product of separable spaces is separable.

> **Thm 2.5 (Hewitt–Marczewski–Pondiczery).** A product of at most $\mathfrak{c} = 2^{\aleph_0}$ separable spaces is separable.

This is remarkable: $\mathbb{R}^{\mathbb{R}}$ (product of $\mathfrak{c}$-many copies of $\mathbb{R}$) is separable, even though it is not first countable!

---

## 6. Permanence Properties

| Property | Subspaces | Products | Quotients |
|----------|:-:|:-:|:-:|
| Separable | Open subspace ✓; arbitrary subspace ✗ | ✓ (countable; up to $\mathfrak{c}$) | ✓ |

**Warning:** A subspace of a separable space need not be separable in general. In metric spaces, every subspace of a separable space is separable (since 2nd countable is hereditary and equivalent to separable).

---

## 7. Separability of Function Spaces

By the **Stone–Weierstrass theorem**, several important function spaces are separable:

- $C([0,1])$ with sup norm: dense countable subset = polynomials with rational coefficients
- $L^p(\mathbb{R}^n)$ for $1 \leq p < \infty$: dense = compactly-supported continuous functions with rational values
- $L^\infty$ is generally **not** separable

---

## 8. Summary Table

| Concept | Details |
|---------|---------|
| Separable | $\exists$ countable dense subset |
| 2nd countable ⟹ separable | Pick one point from each basis element |
| Metric: separable ⟺ 2nd countable | Balls with dense-point centers form basis |
| Products | Up to $\mathfrak{c}$-many factors preserve separability |
| Subspaces | Not hereditary in general; hereditary in metric spaces |
| Key examples | $\mathbb{R}$, $\ell^2$, $C([0,1])$ separable; $\ell^\infty$ not |

---

## 9. Quick Revision Questions

1. Define separability. Give three examples and one non-example.

2. Prove: second countable ⟹ separable.

3. Prove: separable metric space ⟹ second countable.

4. Is every subspace of a separable space separable? Distinguish the general and metric cases.

5. State the Hewitt–Marczewski–Pondiczery theorem. Why is it surprising?

6. Why is $\ell^\infty$ not separable?

---

[← Previous: First and Second Countable](01-first-and-second-countable.md) | [Back to README](../README.md) | [Next: Lindelöf Spaces →](03-lindelof-spaces.md)
