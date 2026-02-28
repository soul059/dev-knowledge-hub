# Chapter 2.1 — Continuous Functions

[← Previous: Comparing Topologies](../01-Topological-Spaces/05-comparing-topologies.md) | [Back to README](../README.md) | [Next: Homeomorphisms →](02-homeomorphisms.md)

---

## 1. Overview

Continuous functions are the **morphisms** of topology — the structure-preserving maps. Unlike in analysis, where continuity is defined via $\epsilon$-$\delta$, topology defines continuity purely in terms of open sets: a function is continuous iff preimages of open sets are open.

---

## 2. Definition of Continuity

> **Def 1.1 (Continuous Function).** Let $(X, \mathcal{T}_X)$ and $(Y, \mathcal{T}_Y)$ be topological spaces. A function $f: X \to Y$ is **continuous** if:
>
> $$\forall\, V \in \mathcal{T}_Y,\quad f^{-1}(V) \in \mathcal{T}_X$$
>
> That is, the preimage of every open set in $Y$ is open in $X$.

```
Continuity Diagram:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    X                          Y
  ┌──────────┐           ┌──────────┐
  │          │    f      │          │
  │ f⁻¹(V)  │ ────────→ │    V     │
  │ (open)   │           │  (open)  │
  │          │           │          │
  └──────────┘           └──────────┘

  V open in Y  ⟹  f⁻¹(V) open in X
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 3. Equivalent Formulations

> **Thm 1.2.** The following are equivalent for $f: (X, \mathcal{T}_X) \to (Y, \mathcal{T}_Y)$:
>
> 1. $f$ is continuous (preimages of opens are open).
> 2. Preimages of closed sets are closed: $C$ closed in $Y$ $\Rightarrow$ $f^{-1}(C)$ closed in $X$.
> 3. For every $A \subseteq X$: $f(\overline{A}) \subseteq \overline{f(A)}$.
> 4. For every $B \subseteq Y$: $\overline{f^{-1}(B)} \subseteq f^{-1}(\overline{B})$.
> 5. For every $x \in X$ and every neighborhood $V$ of $f(x)$ in $Y$, $f^{-1}(V)$ is a neighborhood of $x$ in $X$.

**Proof of (1)⟺(2):** $f^{-1}(Y \setminus C) = X \setminus f^{-1}(C)$. So preimage of complement = complement of preimage. ∎

---

## 4. Continuity via Basis or Subbasis

> **Thm 1.3.** $f: X \to Y$ is continuous if and only if:
>
> $f^{-1}(B)$ is open in $X$ for every $B$ in some **basis** $\mathcal{B}$ (or **subbasis** $\mathcal{S}$) for $\mathcal{T}_Y$.

This is immensely useful: we only need to check preimages of basis elements!

**Application:** To verify $f: \mathbb{R} \to \mathbb{R}$ is continuous, we only check that $f^{-1}((a,b))$ is open for every open interval $(a,b)$.

---

## 5. Continuity at a Point

> **Def 1.4.** $f: X \to Y$ is **continuous at $x_0 \in X$** if for every open $V$ containing $f(x_0)$, there exists an open $U$ containing $x_0$ with $f(U) \subseteq V$.

> **Prop 1.5.** $f$ is continuous (globally) if and only if $f$ is continuous at every point $x \in X$.

```
Continuity at a point:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    X                     Y
  ┌──────────┐       ┌──────────┐
  │  ╭─ U ─╮ │  f    │  ╭─ V ─╮ │
  │  │ •x₀ │ │ ───→  │  │•f(x₀)│ │
  │  ╰─────╯ │       │  ╰─────╯ │
  └──────────┘       └──────────┘

  Given V ∋ f(x₀), find U ∋ x₀ with f(U) ⊆ V
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 6. Operations Preserving Continuity

> **Thm 1.6 (Composition).** If $f: X \to Y$ and $g: Y \to Z$ are continuous, then $g \circ f: X \to Z$ is continuous.
>
> **Proof:** $(g \circ f)^{-1}(W) = f^{-1}(g^{-1}(W))$. Since $g^{-1}(W)$ is open in $Y$ and $f^{-1}$ of open is open, the result follows. ∎

> **Prop 1.7 (Restriction).** If $f: X \to Y$ is continuous and $A \subseteq X$ has the subspace topology, then $f|_A: A \to Y$ is continuous.

> **Prop 1.8 (Pasting Lemma).** Let $X = A \cup B$ where $A, B$ are both closed (or both open). If $f: A \to Y$ and $g: B \to Y$ are continuous and $f|_{A \cap B} = g|_{A \cap B}$, then $h: X \to Y$ defined by
> $$h(x) = \begin{cases} f(x) & x \in A \\ g(x) & x \in B \end{cases}$$
> is continuous.

```
Pasting Lemma:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ┌──── A ────┬──── B ────┐
  │           │           │
  │   f       │ A∩B  g    │
  │           │(agree)    │      ──→  Y
  │           │           │
  └───────────┴───────────┘
        X = A ∪ B

  Result: h = f on A, g on B is continuous
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 7. Continuity and Convergence (Sequences and Nets)

In a **first countable** space, continuity can be characterized by sequences:

> **Prop 1.9.** If $X$ is first countable, then $f: X \to Y$ is continuous at $x$ iff for every sequence $x_n \to x$, $f(x_n) \to f(x)$.

> **⚠️ Warning:** In general topological spaces (not first countable), sequences are insufficient. One needs **nets** or **filters** to characterize continuity.

---

## 8. Important Examples

### 8.1 Constant Functions

$f(x) = c$ for all $x$. Always continuous (preimage of any open set is $\emptyset$ or $X$).

### 8.2 Inclusion Maps

$\iota: A \hookrightarrow X$ where $A$ has the subspace topology. Always continuous: $\iota^{-1}(U) = U \cap A$, which is open in $A$.

### 8.3 The Identity Map

$\text{id}: (X, \mathcal{T}_1) \to (X, \mathcal{T}_2)$ is continuous iff $\mathcal{T}_2 \subseteq \mathcal{T}_1$ (i.e., $\mathcal{T}_1$ is finer than $\mathcal{T}_2$).

### 8.4 Projection Maps

$\pi_i: X_1 \times X_2 \to X_i$ (product topology). Always continuous — this is by design of the product topology (Unit 3).

### 8.5 Characterizing Standard Continuity

For $f: \mathbb{R} \to \mathbb{R}$:
- Topological def: $f^{-1}((a,b))$ is open $\forall\, a < b$.
- This exactly recovers the $\epsilon$-$\delta$ definition from analysis.

---

## 9. Continuous Functions and Topological Properties

A continuous function $f: X \to Y$ maps:

| If $X$ has... | Then $f(X)$ has... | Condition |
|---------------|--------------------|----|
| Compact | Compact | $f$ continuous, onto $f(X)$ |
| Connected | Connected | $f$ continuous, onto $f(X)$ |
| Path-connected | Path-connected | $f$ continuous, onto $f(X)$ |
| Dense subset $D$ | Dense image $f(D)$ in $f(X)$ | $f$ continuous, onto $f(X)$ |

**⚠️** Continuous images do **not** preserve: Hausdorff, metrizable, open, closed (in the ambient space).

---

## 10. The Category **Top**

Topological spaces and continuous functions form a **category**:

- **Objects:** Topological spaces $(X, \mathcal{T})$
- **Morphisms:** Continuous functions $f: X \to Y$
- **Identity:** $\text{id}_X$ (always continuous)
- **Composition:** $g \circ f$ (always continuous)

This categorical viewpoint unifies topology with other branches of mathematics.

---

## 11. Summary Table

| Concept | Definition / Key Point |
|---------|----------------------|
| Continuous $f: X \to Y$ | $f^{-1}(\text{open}) = \text{open}$ |
| Equivalently | Preimage of closed = closed; $f(\overline{A}) \subseteq \overline{f(A)}$ |
| At a point $x_0$ | Every nbhd of $f(x_0)$ pulls back to a nbhd of $x_0$ |
| Basis/subbasis check | Enough to check preimages of basis/subbasis elements |
| Composition | $g \circ f$ continuous if both are |
| Pasting lemma | Glue continuous functions on closed (or open) covers |
| Preserves | Compactness, connectedness, path-connectedness |
| Does not preserve | Hausdorff, openness/closedness of sets, metrizability |

---

## 12. Quick Revision Questions

1. State the topological definition of continuity and prove it is equivalent to the condition that preimages of closed sets are closed.

2. Let $f: \mathbb{R}_\ell \to \mathbb{R}$ be the identity function (from Sorgenfrey to standard). Is it continuous? What about the reverse direction?

3. Prove the composition of two continuous functions is continuous.

4. Give an example showing that a continuous image of a Hausdorff space need not be Hausdorff.

5. State the Pasting Lemma. Why is the hypothesis that $A$ and $B$ are both closed (or both open) necessary?

6. If $f: X \to Y$ is continuous and $D \subseteq X$ is dense, prove that $f(D)$ is dense in $f(X)$ (with the subspace topology).

---

[← Previous: Comparing Topologies](../01-Topological-Spaces/05-comparing-topologies.md) | [Back to README](../README.md) | [Next: Homeomorphisms →](02-homeomorphisms.md)
