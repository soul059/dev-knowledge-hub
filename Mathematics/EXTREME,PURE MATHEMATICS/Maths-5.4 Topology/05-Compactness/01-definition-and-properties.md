# Chapter 5.1 — Compactness: Definition and Properties

[← Previous: Local Connectedness](../04-Connectedness/04-local-connectedness.md) | [Back to README](../README.md) | [Next: Compact Subsets of ℝⁿ →](02-compact-subsets-of-Rn.md)

---

## 1. Overview

**Compactness** is one of the most powerful concepts in topology. It is an abstract generalization of "closed and bounded" (from $\mathbb{R}^n$), capturing the idea that a space is "finite-like" in a topological sense: every open cover has a finite subcover. Compact spaces have extraordinarily nice properties.

---

## 2. Definition

> **Def 1.1 (Open Cover).** An **open cover** of $X$ is a collection $\{U_\alpha\}_{\alpha \in A}$ of open sets with $X = \bigcup_\alpha U_\alpha$. A **subcover** is a subcollection that still covers $X$.

> **Def 1.2 (Compact).** $X$ is **compact** if every open cover of $X$ has a **finite subcover**.

$$\forall \{U_\alpha\}_{\alpha \in A} \text{ open cover of } X,\;\; \exists \alpha_1, \ldots, \alpha_n \in A : X = U_{\alpha_1} \cup \cdots \cup U_{\alpha_n}$$

```
Compactness Idea:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Given: infinitely many open sets covering X

  ╭──╮╭───╮╭──╮╭────╮╭─╮╭──╮╭───╮
  │U₁││U₂ ││U₃││ U₄ ││U₅││U₆││...│   (open cover)
  ╰──╯╰───╯╰──╯╰────╯╰─╯╰──╯╰───╯

  Compact ⟹ can select finitely many:

  ╭──╮     ╭──╮╭────╮     ╭───╮
  │U₁│     │U₃││ U₄ │     │U₇ │        (finite subcover)
  ╰──╯     ╰──╯╰────╯     ╰───╯
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 3. Basic Examples

| Space | Compact? | Reason |
|-------|:---:|------|
| $[a,b] \subset \mathbb{R}$ | ✓ | Heine-Borel theorem |
| $(0,1) \subset \mathbb{R}$ | ✗ | Cover $\{(1/n, 1)\}$ has no finite subcover |
| $\mathbb{R}$ | ✗ | Cover $\{(-n, n)\}$ has no finite subcover |
| Finite set (any topology) | ✓ | Any cover of a finite set has a finite subcover |
| Indiscrete space | ✓ | Only cover: $\{X\}$ |
| Discrete infinite set | ✗ | Singletons form a cover with no finite subcover |
| Cofinite topology on infinite $X$ | ✓ | Any open set misses finitely many points |

---

## 4. Fundamental Properties

### 4.1 Continuous Image of Compact Is Compact

> **Thm 1.3.** If $f: X \to Y$ is continuous and $X$ is compact, then $f(X)$ is compact.

**Proof:** Let $\{V_\alpha\}$ cover $f(X)$. Then $\{f^{-1}(V_\alpha)\}$ covers $X$. Extract finite subcover $f^{-1}(V_{\alpha_1}), \ldots, f^{-1}(V_{\alpha_n})$. Then $V_{\alpha_1}, \ldots, V_{\alpha_n}$ covers $f(X)$. ∎

> **Cor 1.4 (Extreme Value Theorem).** A continuous function $f: X \to \mathbb{R}$ on a compact space $X$ achieves its maximum and minimum.

### 4.2 Closed Subset of Compact Is Compact

> **Thm 1.5.** If $X$ is compact and $C \subseteq X$ is closed, then $C$ is compact.

**Proof:** Let $\{U_\alpha\}$ cover $C$. Then $\{U_\alpha\} \cup \{X \setminus C\}$ covers $X$. Extract finite subcover. Remove $X \setminus C$ if present. The remaining sets cover $C$. ∎

### 4.3 Compact Subset of Hausdorff Is Closed

> **Thm 1.6.** If $X$ is Hausdorff and $K \subseteq X$ is compact, then $K$ is closed.

**Proof sketch:** Show $X \setminus K$ is open. For $x \notin K$ and each $k \in K$, use Hausdorff to separate $x$ and $k$ by open sets $U_k \ni x$, $V_k \ni k$. The $V_k$'s cover $K$; extract finite subcover $V_{k_1}, \ldots, V_{k_n}$. Then $U_{k_1} \cap \cdots \cap U_{k_n}$ is an open neighborhood of $x$ disjoint from $K$. ∎

### 4.4 Compact Hausdorff: Closed ⟺ Compact

> **Cor 1.7.** In a compact Hausdorff space: $K$ is compact $\iff$ $K$ is closed.

---

## 5. Compactness and Continuity (Powerful Consequences)

> **Thm 1.8.** A continuous bijection from a compact space to a Hausdorff space is a **homeomorphism**.

**Proof:** We need $f^{-1}$ continuous, i.e., $f$ maps closed sets to closed sets. Closed subsets of compact $X$ are compact (Thm 1.5). Continuous images of compact sets are compact (Thm 1.3). Compact subsets of Hausdorff $Y$ are closed (Thm 1.6). ∎

This is enormously useful — it means we need only check bijectivity and continuity!

---

## 6. The Finite Intersection Property (FIP)

> **Def 1.9.** A collection $\{C_\alpha\}$ of sets has the **finite intersection property (FIP)** if every finite subcollection has nonempty intersection.

> **Thm 1.10 (FIP Characterization).** $X$ is compact iff for every collection of **closed** sets $\{C_\alpha\}$ with the FIP, $\bigcap_\alpha C_\alpha \neq \emptyset$.

**Proof:** Take complements of the open cover characterization. ∎

```
FIP ↔ Compactness (dual viewpoint):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  OPEN COVER version:
    Every open cover has finite subcover

  CLOSED SET version (dual):
    If closed sets have FIP, their
    total intersection is nonempty

  These are logically equivalent
  (De Morgan duality)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 7. Tychonoff's Theorem

> **Thm 1.11 (Tychonoff).** The product $\prod_\alpha X_\alpha$ (with the product topology) is compact if and only if each $X_\alpha$ is compact.

This is one of the deepest theorems in general topology. The "$\Rightarrow$" direction is easy (projections are continuous). The "$\Leftarrow$" direction requires the Axiom of Choice (typically via Zorn's Lemma or Alexander's Subbasis Lemma).

> **✦ Key Insight:** Tychonoff's theorem is equivalent to the Axiom of Choice!

---

## 8. Tube Lemma

> **Lem 1.12 (Tube Lemma).** Let $X \times Y$ be a product with $Y$ compact. If $U$ is an open set in $X \times Y$ containing the "slice" $\{x_0\} \times Y$, then $U$ contains a "tube" $W \times Y$ for some open $W \ni x_0$ in $X$.

```
Tube Lemma:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Y (compact)
  ▲  ┌────────────────────┐
  │  │   ╭── U ─────╮     │
  │  │   │   ████████│     │
  │  │   │   █tube██│     │
  │  │   │   ████████│     │
  │  │   ╰───────────╯     │
  │  └──────────|──────────┘
  └──────────── x₀ ─────────→ X
            ├─W─┤
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

The Tube Lemma is used to prove that $X \times Y$ is compact when both factors are.

---

## 9. Summary Table

| Concept | Key Point |
|---------|-----------|
| Compact | Every open cover has finite subcover |
| Equivalent (FIP) | Closed sets with FIP have nonempty intersection |
| Continuous image | Compact → compact |
| Closed ⊂ compact | Compact |
| Compact ⊂ Hausdorff | Closed |
| Compact + Hausdorff | Closed ⟺ compact subsets |
| Compact → Hausdorff bijection | Homeomorphism |
| EVT | Continuous $f: X \to \mathbb{R}$, $X$ compact ⟹ $f$ achieves max/min |
| Tychonoff | $\prod X_\alpha$ compact ⟺ each $X_\alpha$ compact |
| Tube Lemma | Slice in open set ⟹ tube neighborhood ($Y$ compact) |

---

## 10. Quick Revision Questions

1. State the definition of compactness and prove that $[0,1]$ is compact (or cite the Heine-Borel theorem).

2. Show that $(0,1)$ is not compact by exhibiting an open cover with no finite subcover.

3. Prove: a continuous image of a compact space is compact.

4. Prove: a compact subset of a Hausdorff space is closed.

5. State the Finite Intersection Property characterization of compactness and prove it is equivalent to the open cover definition.

6. State Tychonoff's theorem. Why is it remarkable for infinite products?

---

[← Previous: Local Connectedness](../04-Connectedness/04-local-connectedness.md) | [Back to README](../README.md) | [Next: Compact Subsets of ℝⁿ →](02-compact-subsets-of-Rn.md)
