# Chapter 6.2 — Regular and Normal Spaces

[← Previous: Hausdorff and T₁ Spaces](01-hausdorff-and-T1-spaces.md) | [Back to README](../README.md) | [Next: Urysohn's Lemma →](03-urysohns-lemma.md)

---

## 1. Overview

**Regular** and **normal** spaces strengthen the Hausdorff axiom by allowing separation of points from closed sets, or closed sets from each other, by disjoint open sets. Normality is the gateway to Urysohn's Lemma and the Tietze Extension Theorem.

---

## 2. Definitions

### 2.1 Regular (T₃)

> **Def 2.1.** $X$ is **regular** if it is T₁ and: for every closed set $F$ and point $x \notin F$, there exist disjoint open sets $U, V$ with $x \in U$ and $F \subseteq V$.

```
Regular: Separate point from closed set
┌────────────── X ──────────────┐
│                                │
│  ┌── U ──┐    ┌─── V ───┐    │
│  │       │    │          │    │
│  │  • x  │    │  F████   │    │
│  │       │    │   ████   │    │
│  └───────┘    └──────────┘    │
│        U ∩ V = ∅              │
└────────────────────────────────┘
```

### 2.2 Normal (T₄)

> **Def 2.2.** $X$ is **normal** if it is T₁ and: for any two disjoint closed sets $F_1, F_2$, there exist disjoint open sets $U_1, U_2$ with $F_1 \subseteq U_1$ and $F_2 \subseteq U_2$.

```
Normal: Separate closed sets from each other
┌────────────── X ──────────────┐
│                                │
│  ┌── U₁ ──┐  ┌── U₂ ──┐     │
│  │         │  │         │     │
│  │  F₁██  │  │  ██F₂   │     │
│  │   ███  │  │  ███    │     │
│  └────────┘  └─────────┘     │
│       U₁ ∩ U₂ = ∅            │
└────────────────────────────────┘
```

### 2.3 Completely Regular (T₃½ / Tychonoff)

> **Def 2.3.** $X$ is **completely regular** if it is T₁ and: for every closed $F$ and $x \notin F$, there exists a continuous $f : X \to [0,1]$ with $f(x) = 0$ and $f|_F \equiv 1$.

---

## 3. The Complete Hierarchy

```
T₄ (Normal) ──→ T₃½ (Completely Regular) ──→ T₃ (Regular) ──→ T₂ ──→ T₁ ──→ T₀
                                                                 
Key: Each implies the one to its right, not vice versa.
```

| Separation | What is separated | By what |
|-----------|-------------------|---------|
| T₂ | Point from point | Disjoint open sets |
| T₃ | Point from closed set | Disjoint open sets |
| T₃½ | Point from closed set | Continuous function |
| T₄ | Closed set from closed set | Disjoint open sets |

---

## 4. Key Theorems

### 4.1 Compact Hausdorff ⟹ Normal

> **Thm 2.4.** Every compact Hausdorff space is normal.

*Proof:*
1. **Compact Hausdorff ⟹ Regular:** Let $x \notin F$ (closed). For each $y \in F$, separate $x$ from $y$ by disjoint open $U_y, V_y$. The $V_y$'s cover $F$; $F$ is closed in compact, so compact; extract finite subcover $V_{y_1}, \ldots, V_{y_n}$. Take $U = \bigcap U_{y_i}$ and $V = \bigcup V_{y_j}$.
2. **Compact regular ⟹ Normal:** Repeat the argument with $F_1$ closed (hence compact) replacing the point $x$. ∎

### 4.2 Metric Spaces Are Normal

> **Thm 2.5.** Every metrizable space is normal.

*Proof:* Given disjoint closed $F_1, F_2$, define
$$U_1 = \bigcup_{x \in F_1} B\bigl(x, \tfrac{1}{2}d(x, F_2)\bigr), \qquad U_2 = \bigcup_{y \in F_2} B\bigl(y, \tfrac{1}{2}d(y, F_1)\bigr)$$
Then $U_1, U_2$ are open, $F_i \subseteq U_i$, and $U_1 \cap U_2 = \emptyset$ (by triangle inequality). ∎

### 4.3 Second Countable + Regular ⟹ Normal

> **Thm 2.6.** A second-countable regular space is normal (and in fact metrizable, by the Urysohn metrization theorem).

---

## 5. Equivalent Formulations of Normality

> **Thm 2.7.** For a T₁ space $X$, the following are equivalent:
> 1. $X$ is normal.
> 2. For every closed $F$ and open $U \supseteq F$, there exists open $V$ with $F \subseteq V \subseteq \overline{V} \subseteq U$.
> 3. (Urysohn's Lemma) For disjoint closed $A, B$, there is continuous $f : X \to [0,1]$ with $f|_A = 0$, $f|_B = 1$.

The equivalence (1) ⟺ (3) is **Urysohn's Lemma** (next chapter).

---

## 6. Important Examples

| Space | Regular | Normal |
|-------|:-------:|:------:|
| $\mathbb{R}^n$ (standard) | ✓ | ✓ |
| Metric spaces | ✓ | ✓ |
| Compact Hausdorff | ✓ | ✓ |
| Sorgenfrey line $\mathbb{R}_\ell$ | ✓ | ✓ |
| $\mathbb{R}_\ell \times \mathbb{R}_\ell$ | ✓ | ✗ |
| $\omega_1 \times (\omega_1 + 1)$ | ✓ | ✗ |
| Cofinite topology (infinite) | ✗ (T₁, not T₂) | ✗ |

**Notable:** The Sorgenfrey plane $\mathbb{R}_\ell^2$ is regular but **not** normal — showing normality is not preserved by products.

---

## 7. Permanence Properties

| Property | Subspace | Product | Quotient |
|----------|:--------:|:-------:|:--------:|
| Regular | ✓ | ✓ | ✗ |
| Completely Regular | ✓ | ✓ | ✗ |
| Normal | Closed subspace ✓ | ✗ in general | ✗ |

**Warning:** Normality is only hereditary for **closed** subspaces. Open or arbitrary subspaces of normal spaces need not be normal.

---

## 8. Summary Table

| Concept | Separates | Method | Implied by |
|---------|-----------|--------|------------|
| Regular (T₃) | Point ↔ closed set | Open sets | Normality, complete regularity |
| Normal (T₄) | Closed ↔ closed | Open sets | Compact Hausdorff, metrizable |
| Completely Regular | Point ↔ closed set | Continuous function | Normality (Urysohn) |
| Compact Hausdorff | — | — | Is always T₄ |
| Metric | — | — | Is always T₄ |

---

## 9. Quick Revision Questions

1. State the definitions of regular, completely regular, and normal spaces.

2. Prove that every compact Hausdorff space is normal.

3. Prove that every metric space is normal using the distance function.

4. Give an example of a space that is regular but not normal.

5. Is normality preserved under products? Under taking subspaces?

6. State the equivalent formulations of normality (including the "nesting" characterisation).

---

[← Previous: Hausdorff and T₁ Spaces](01-hausdorff-and-T1-spaces.md) | [Back to README](../README.md) | [Next: Urysohn's Lemma →](03-urysohns-lemma.md)
