# Chapter 2.2 — Homeomorphisms

[← Previous: Continuous Functions](01-continuous-functions.md) | [Back to README](../README.md) | [Next: Topological Properties →](03-topological-properties.md)

---

## 1. Overview

A **homeomorphism** is the correct notion of "sameness" in topology. Two spaces are homeomorphic if one can be continuously deformed into the other (and back). Any property preserved by homeomorphisms is a **topological property** — the central objects of study.

---

## 2. Definition

> **Def 2.1 (Homeomorphism).** A function $f: X \to Y$ is a **homeomorphism** if:
>
> 1. $f$ is a **bijection** (one-to-one and onto).
> 2. $f$ is **continuous**.
> 3. $f^{-1}: Y \to X$ is **continuous**.
>
> Write $X \cong Y$ if such an $f$ exists.

> **⚠️ Warning:** A continuous bijection need **not** be a homeomorphism! The inverse may fail to be continuous.

---

## 3. Equivalent Characterization

> **Prop 2.2.** A bijection $f: X \to Y$ is a homeomorphism if and only if:
>
> $$U \text{ is open in } X \iff f(U) \text{ is open in } Y$$
>
> That is, $f$ induces a **bijection between the topologies** $\mathcal{T}_X$ and $\mathcal{T}_Y$.

**Proof:** $f$ continuous means $f^{-1}(V)$ open $\Rightarrow$ open preimages. $f^{-1}$ continuous means $(f^{-1})^{-1}(U) = f(U)$ open, i.e., images of opens are open. ∎

```
Homeomorphism = bijection on points AND on open sets:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    (X, 𝒯_X)     f (bijection)     (Y, 𝒯_Y)
  ┌──────────┐    ──────────→    ┌──────────┐
  │ points   │    ←──────────    │ points   │
  │ of X     │      f⁻¹         │ of Y     │
  └──────────┘                   └──────────┘

  𝒯_X ──────── 1:1 correspondence ──────── 𝒯_Y
    U ∈ 𝒯_X ←→ f(U) ∈ 𝒯_Y
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 4. Classic Examples

### 4.1 $(0,1) \cong \mathbb{R}$

$$f(x) = \tan\!\left(\pi x - \frac{\pi}{2}\right)$$

This is a continuous bijection with continuous inverse (from calculus). So the bounded interval $(0,1)$ is homeomorphic to all of $\mathbb{R}$.

```
       f: (0,1) → ℝ
    1 ┤              ╱
      │            ╱
  0.5 ┤──────────●
      │        ╱
    0 ┤──────╱─────────────
      └──┬──┬──┬──┬──┬──→ ℝ
        -∞ -1  0  1  +∞
```

### 4.2 $[0,1] \not\cong (0,1)$

These are **not** homeomorphic: $[0,1]$ is compact, $(0,1)$ is not. Compactness is a topological property.

### 4.3 $S^1 \not\cong \mathbb{R}$

The circle is compact; $\mathbb{R}$ is not. Also, removing a point from $S^1$ leaves a connected space; removing a point from $\mathbb{R}$ disconnects it. *(This latter argument uses invariance of connectedness.)*

### 4.4 Coffee Cup $\cong$ Donut (Torus)

The classic visual: a coffee cup (with one handle) is homeomorphic to a torus. Both have "one hole."

```
Coffee Cup ≅ Torus (one hole each):

   ╭───╮              ╭─────────╮
  │     │  handle     │ ╭─────╮ │
  │  ╭──╯──╮         │ │     │ │
  │  │      │         │ │     │ │
  │  ╰──╮──╯         │ ╰─────╯ │
  │     │             ╰─────────╯
  ╰─────╯
  Coffee cup           Torus
```

### 4.5 Continuous Bijection That Is NOT a Homeomorphism

Let $f: [0, 2\pi) \to S^1$ defined by $f(t) = (\cos t, \sin t)$.

- $f$ is continuous and bijective.
- But $f^{-1}$ is **not** continuous: as you approach the point $(1,0)$ from "below" on $S^1$, the preimage jumps from near $2\pi$ to $0$.

This shows why the inverse being continuous is a separate requirement.

---

## 5. Topological Embedding

> **Def 2.3 (Embedding).** A function $f: X \to Y$ is a **topological embedding** if $f: X \to f(X)$ is a homeomorphism (where $f(X)$ has the subspace topology from $Y$).

Equivalently: $f$ is injective, continuous, and open (or closed) onto its image.

**Example:** $\iota: (0,1) \hookrightarrow \mathbb{R}$ by $\iota(x) = x$ is an embedding.

---

## 6. Invariants for Distinguishing Spaces

To prove $X \not\cong Y$, find a **topological invariant** — a property preserved by homeomorphism — that differs:

| Invariant | $X$ | $Y$ | Conclusion |
|-----------|-----|-----|------------|
| Compactness | $[0,1]$: compact | $(0,1)$: not compact | $[0,1] \not\cong (0,1)$ |
| Cardinality | $\mathbb{R}$: uncountable | $\mathbb{Q}$: countable | $\mathbb{R} \not\cong \mathbb{Q}$ |
| Connectedness | $\mathbb{R}$: connected | $\mathbb{Q}$: totally disconnected | $\mathbb{R} \not\cong \mathbb{Q}$ |
| Remove a point | $\mathbb{R} \setminus \{0\}$: 2 components | $\mathbb{R}^2 \setminus \{0\}$: 1 component | $\mathbb{R} \not\cong \mathbb{R}^2$ |
| Fundamental group | $S^1$: $\pi_1 = \mathbb{Z}$ | $\mathbb{R}^2$: $\pi_1 = 0$ | $S^1 \not\cong \mathbb{R}^2$ |

---

## 7. The Homeomorphism Relation

> **Prop 2.4.** $\cong$ is an equivalence relation on topological spaces:
>
> 1. **Reflexive:** $\text{id}_X: X \to X$ is a homeomorphism.
> 2. **Symmetric:** If $f: X \to Y$ is a homeomorphism, so is $f^{-1}: Y \to X$.
> 3. **Transitive:** If $f: X \to Y$ and $g: Y \to Z$ are homeomorphisms, so is $g \circ f$.

---

## 8. Local Homeomorphism

> **Def 2.5.** A function $f: X \to Y$ is a **local homeomorphism** if every $x \in X$ has an open neighborhood $U$ such that $f(U)$ is open in $Y$ and $f|_U: U \to f(U)$ is a homeomorphism.

**Example:** $f: \mathbb{R} \to S^1$ by $f(t) = e^{2\pi i t}$ is a local homeomorphism but **not** a homeomorphism (not injective).

> **✦ Key Insight:** Every homeomorphism is a local homeomorphism, but not conversely.

---

## 9. Summary Table

| Concept | Definition / Key Point |
|---------|----------------------|
| Homeomorphism | Continuous bijection with continuous inverse |
| $X \cong Y$ | $X$ homeomorphic to $Y$; "topologically the same" |
| Not enough | Continuous bijection ≠ homeomorphism (inverse may fail) |
| Embedding | Homeomorphism onto image (subspace topology) |
| Topological invariant | Property preserved by homeomorphisms |
| Local homeomorphism | Locally a homeomorphism at every point |
| Equivalence relation | $\cong$ is reflexive, symmetric, transitive |

---

## 10. Quick Revision Questions

1. Give a continuous bijection that is **not** a homeomorphism. Explain where the inverse fails to be continuous.

2. Prove that compactness is a topological invariant (i.e., if $X \cong Y$ and $X$ is compact, then $Y$ is compact).

3. Is $\mathbb{R}^2 \cong \mathbb{R}$? What topological invariant distinguishes them?

4. Show that $(a,b) \cong (c,d)$ for any real intervals with $a < b$, $c < d$.

5. Define topological embedding and give an example of a continuous injection that is **not** an embedding.

6. Prove that homeomorphism is an equivalence relation.

---

[← Previous: Continuous Functions](01-continuous-functions.md) | [Back to README](../README.md) | [Next: Topological Properties →](03-topological-properties.md)
