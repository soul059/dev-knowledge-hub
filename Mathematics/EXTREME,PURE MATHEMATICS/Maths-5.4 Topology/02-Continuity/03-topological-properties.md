# Chapter 2.3 — Topological Properties

[← Previous: Homeomorphisms](02-homeomorphisms.md) | [Back to README](../README.md) | [Next: Open and Closed Maps →](04-open-and-closed-maps.md)

---

## 1. Overview

A **topological property** (or **topological invariant**) is any property of a topological space that is preserved by homeomorphisms. These invariants are the primary tools for classifying spaces — if two spaces differ on any invariant, they cannot be homeomorphic.

---

## 2. Formal Definition

> **Def 3.1 (Topological Property).** A property $P$ of topological spaces is a **topological property** if:
>
> Whenever $(X, \mathcal{T}_X)$ has property $P$ and $f: X \to Y$ is a homeomorphism, then $(Y, \mathcal{T}_Y)$ also has property $P$.

Equivalently: $P$ depends only on the topology (the collection of open sets), not on any specific representation.

---

## 3. Catalogue of Topological Properties

### 3.1 Properties We Will Study in Detail (Later Units)

| Property | Preserved by Homeomorphism? | Preserved by Continuous Image? |
|----------|:---------------------------:|:------------------------------:|
| Compactness | ✓ | ✓ |
| Connectedness | ✓ | ✓ |
| Path-connectedness | ✓ | ✓ |
| Hausdorff (T₂) | ✓ | ✗ |
| T₁ | ✓ | ✗ |
| Regular (T₃) | ✓ | ✗ |
| Normal (T₄) | ✓ | ✗ |
| First countable | ✓ | ✗ |
| Second countable | ✓ | ✗ |
| Separable | ✓ | ✓ |
| Metrizable | ✓ | ✗ |
| Locally compact | ✓ | ✗ |
| Locally connected | ✓ | ✗ |

### 3.2 Properties That Are NOT Topological

| Property | Why Not Topological? |
|----------|---------------------|
| Boundedness | $(0,1) \cong \mathbb{R}$, but $(0,1)$ bounded, $\mathbb{R}$ unbounded |
| Completeness (metric) | $(0,1)$ not complete, $\mathbb{R}$ complete, yet $(0,1) \cong \mathbb{R}$ |
| Having diameter $< 1$ | Changes under homeomorphism |
| Being a subset of $\mathbb{R}^n$ | Extrinsic, not intrinsic |

```
Topological Properties — The Filter:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  All properties of spaces
  ┌────────────────────────────┐
  │  Diameter, boundedness,    │  ← NOT topological
  │  specific metric values    │    (fail the homeomorphism test)
  │ ┌────────────────────────┐ │
  │ │  Compactness,          │ │  ← TOPOLOGICAL
  │ │  connectedness,        │ │    (pass through homeomorphisms)
  │ │  separation axioms,    │ │
  │ │  countability axioms   │ │
  │ └────────────────────────┘ │
  └────────────────────────────┘
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 4. Using Invariants to Distinguish Spaces

**Strategy:** To prove $X \not\cong Y$, find a topological property that $X$ has but $Y$ does not.

### Worked Example 1: $\mathbb{R} \not\cong \mathbb{R}^2$

**Approach:** Remove a point and compare the resulting spaces.

- $\mathbb{R} \setminus \{0\}$ has **2** connected components: $(-\infty, 0)$ and $(0, \infty)$.
- $\mathbb{R}^2 \setminus \{(0,0)\}$ is **connected** (path-connected, even).

If $\mathbb{R} \cong \mathbb{R}^2$ via homeomorphism $f$, then $\mathbb{R} \setminus \{0\} \cong \mathbb{R}^2 \setminus \{f(0)\}$. But the left side has 2 components and the right has 1. Contradiction. ∎

### Worked Example 2: $[0,1] \not\cong S^1$

- Removing an interior point from $[0,1]$ (say $1/2$) disconnects it into $[0, 1/2) \cup (1/2, 1]$.
- Removing any point from $S^1$ leaves it connected (homeomorphic to $\mathbb{R}$).

So $[0,1] \not\cong S^1$. ∎

### Worked Example 3: $(0,1) \not\cong [0,1)$

Remove $0$:
- $(0,1)$ without a point remains a single interval minus a point → 2 components.
- $[0,1)$ without $0$ becomes $(0,1)$ → 1 component.

But we must be more careful: in $(0,1)$, some points when removed give 2 components. In $[0,1)$, removing $0$ gives 1 component — but removing any interior point also gives 2 components. So the argument is: in $[0,1)$, there exists a point (namely $0$) whose removal does not disconnect the space. In $(0,1)$, removal of **any** point disconnects. These properties are topological, so the spaces are not homeomorphic. ∎

---

## 5. Cardinality-Based Invariants

| Invariant | Definition |
|-----------|-----------|
| $|X|$ | Cardinality of the underlying set |
| Number of open sets $|\mathcal{T}|$ | Size of the topology |
| Number of connected components | Partition into maximal connected subsets |
| Number of isolated points | Points $x$ where $\{x\}$ is open |

These are all topological invariants.

---

## 6. Algebraic Invariants (Preview)

More powerful invariants come from algebraic topology (Unit 9):

| Invariant | Description |
|-----------|-------------|
| $\pi_1(X, x_0)$ | Fundamental group — counts "holes" |
| $H_n(X)$ | Homology groups — higher-dimensional holes |
| $\pi_n(X, x_0)$ | Higher homotopy groups |
| Euler characteristic $\chi$ | Alternating sum of Betti numbers |

**Example:** $\pi_1(S^1) = \mathbb{Z}$ but $\pi_1(\mathbb{R}^2) = 0$, so $S^1 \not\cong \mathbb{R}^2$.

---

## 7. Hereditary and Productive Properties

> **Def 3.2.** A property $P$ is **hereditary** if: $X$ has $P$ $\Rightarrow$ every subspace of $X$ has $P$.

> **Def 3.3.** A property $P$ is **productive** if: each $X_\alpha$ has $P$ $\Rightarrow$ $\prod X_\alpha$ has $P$.

| Property | Hereditary? | Productive (finite)? | Productive (arbitrary)? |
|----------|:-----------:|:-------------------:|:----------------------:|
| Hausdorff | ✓ | ✓ | ✓ |
| Compact | ✗ (closed subspaces only) | ✓ | ✓ (Tychonoff) |
| Connected | ✗ | ✓ | ✓ |
| Second countable | ✓ | ✓ | ✗ |
| Metrizable | ✓ | ✓ | ✗ (in general) |
| T₁ | ✓ | ✓ | ✓ |

---

## 8. Classification Results

Some complete classifications are known:

| Spaces | Classification Theorem |
|--------|----------------------|
| Compact surfaces | Classified by orientability + genus (classification of surfaces) |
| Countable compact metric | Classified up to homeomorphism by Cantor-Bendixson rank |
| Second countable compact Hausdorff zero-dimensional | Classification by Brouwer's theorem (= Cantor set if perfect) |

---

## 9. Summary Table

| Concept | Key Point |
|---------|-----------|
| Topological property | Preserved by homeomorphisms |
| Examples | Compactness, connectedness, Hausdorff, countability, etc. |
| Non-examples | Boundedness, completeness, diameter |
| Strategy to distinguish | Find invariant that differs |
| Point-removal argument | Remove point(s) and compare resulting properties |
| Hereditary | Passes to all subspaces |
| Productive | Passes to products |
| Algebraic invariants | Fundamental group $\pi_1$, homology $H_n$, etc. |

---

## 10. Quick Revision Questions

1. Give three examples of topological properties and two examples of properties that are **not** topological.

2. Prove that $\mathbb{R} \not\cong \mathbb{R}^2$ using a point-removal argument.

3. Is "having exactly 5 open sets" a topological property? Why or why not?

4. Show that the number of connected components is a topological invariant.

5. Define what it means for a property to be **hereditary**. Is compactness hereditary? Is the Hausdorff property hereditary?

6. Why is completeness of a metric space **not** a topological property? Give a specific counterexample.

---

[← Previous: Homeomorphisms](02-homeomorphisms.md) | [Back to README](../README.md) | [Next: Open and Closed Maps →](04-open-and-closed-maps.md)
