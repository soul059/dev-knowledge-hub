# Chapter 6.3 — Urysohn's Lemma

[← Previous: Regular and Normal Spaces](02-regular-and-normal-spaces.md) | [Back to README](../README.md) | [Next: Tietze Extension Theorem →](04-tietze-extension-theorem.md)

---

## 1. Overview

**Urysohn's Lemma** is one of the deepest results about normal spaces. It guarantees the existence of continuous real-valued functions separating disjoint closed sets — bridging topology and analysis.

---

## 2. Statement

> **Theorem 3.1 (Urysohn's Lemma).** Let $X$ be a normal space and $A, B \subseteq X$ disjoint closed sets. Then there exists a continuous function $f : X \to [0,1]$ such that
> $$f(x) = 0 \;\;\forall x \in A, \qquad f(x) = 1 \;\;\forall x \in B.$$

Such an $f$ is called a **Urysohn function** for the pair $(A, B)$.

```
  f : X → [0, 1]
  ┌────────────────────────────────────┐
  │  A██████       ····       ██████B  │
  │  f = 0    0 < f < 1       f = 1   │
  └────────────────────────────────────┘

  Graph of f along a "path" from A to B:
  1 ─────────────────────────────████
  │                           ···
  │                       ···
  │                   ···
  │               ···
  │           ···
  0 ████···
    A ─────────────────────────── B
```

---

## 3. Proof (Construction via Dyadic Rationals)

### Step 1: Build a Family of Open Sets

Let $D = \{r \in \mathbb{Q} : 0 \leq r \leq 1, \; r = p/2^n\}$ (dyadic rationals in $[0,1]$).

Start with $U_1 = X \setminus B$ (open, contains $A$) and $U_0$: by normality, $\exists\, U_0$ open with $A \subseteq U_0 \subseteq \overline{U_0} \subseteq U_1$.

**Inductive construction:** For each dyadic rational $r < s$ already assigned, insert $U_t$ for $t$ between $r$ and $s$ using normality:

$$\overline{U_r} \subseteq U_t \subseteq \overline{U_t} \subseteq U_s$$

```
Nesting chain (for some dyadic rationals):
A ⊆ U₀ ⊆ U̅₀ ⊆ U_{1/4} ⊆ U̅_{1/4} ⊆ U_{1/2} ⊆ U̅_{1/2} ⊆ U_{3/4} ⊆ U̅_{3/4} ⊆ U₁ = X\B
```

### Step 2: Define $f$

$$f(x) = \begin{cases} \inf\{r \in D : x \in U_r\} & \text{if } x \in U_1 \\ 1 & \text{if } x \notin U_1 \end{cases}$$

### Step 3: Verify Properties

- $f(A) = \{0\}$: $A \subseteq U_0 \subseteq U_r$ for all $r \geq 0$, so $\inf = 0$.
- $f(B) = \{1\}$: $B \cap U_1 = \emptyset$, so $f = 1$ on $B$.
- **Continuity:** For $a < f(x) < b$, choose dyadic $r, s$ with $a < r < f(x) < s < b$. Then $U_s \setminus \overline{U_r}$ is an open neighbourhood of $x$ mapping into $(a,b)$. ∎

---

## 4. Converse Direction

> **Thm 3.2.** If for every pair of disjoint closed sets $A, B$ in a T₁ space $X$ there exists a Urysohn function, then $X$ is normal.

*Proof:* Given the function $f$, set $U = f^{-1}([0, 1/2))$ and $V = f^{-1}((1/2, 1])$. These are disjoint open sets separating $A$ and $B$. ∎

Thus: **$X$ is normal ⟺ Urysohn functions exist for all disjoint closed pairs.**

---

## 5. Important Consequences

### 5.1 Urysohn Metrization Theorem

> **Thm 3.3.** A second-countable T₃ (regular Hausdorff) space is metrizable.

*Proof idea:* Second-countable + regular ⟹ normal. Use Urysohn functions for countably many pairs to embed $X$ into $\mathbb{R}^\omega$ (or Hilbert cube $[0,1]^\omega$). The embedding inherits a metric.

### 5.2 Completely Regular Embedding

Normal spaces are completely regular: the Urysohn function itself witnesses complete regularity.

### 5.3 Partition of Unity (Preview)

On normal spaces (especially paracompact ones), Urysohn-type constructions yield **partitions of unity** subordinate to open covers.

---

## 6. Key Subtlety

Urysohn's Lemma does **NOT** say that $f^{-1}(0) = A$. The function can be zero outside $A$:

$$A \subseteq f^{-1}(0) \subseteq X$$

To get $f^{-1}(0) = A$ exactly, you need $A$ to be a $G_\delta$ set (countable intersection of open sets) — this is related to **perfectly normal** spaces.

---

## 7. Worked Example

**Example:** $X = \mathbb{R}$, $A = (-\infty, 0]$, $B = [1, \infty)$.

A Urysohn function can be constructed explicitly:

$$f(x) = \begin{cases} 0 & x \leq 0 \\ x & 0 \leq x \leq 1 \\ 1 & x \geq 1 \end{cases}$$

This is continuous, $f|_A = 0$, $f|_B = 1$.

In general normal spaces, we can't write such explicit formulas — the dyadic construction is needed.

---

## 8. Summary Table

| Concept | Details |
|---------|---------|
| Urysohn's Lemma | Normal ⟹ $\exists$ continuous $f$ separating disjoint closed sets |
| Proof technique | Dyadic rationals, nested open sets via normality |
| Converse | Existence of Urysohn functions ⟹ normality |
| $f^{-1}(0) \supseteq A$ | Equality requires perfectly normal |
| Metrization | Second-countable + regular ⟹ metrizable (via Urysohn) |
| Significance | Bridges topology and analysis; foundation for Tietze |

---

## 9. Quick Revision Questions

1. State Urysohn's Lemma precisely.

2. Outline the proof. What role do dyadic rationals play?

3. Why is normality essential? Give an example where the lemma fails without it.

4. Prove the converse: Urysohn functions ⟹ normality.

5. State the Urysohn Metrization Theorem and explain how Urysohn's Lemma is used in its proof.

6. Does Urysohn's Lemma guarantee $f^{-1}(0) = A$? Explain.

---

[← Previous: Regular and Normal Spaces](02-regular-and-normal-spaces.md) | [Back to README](../README.md) | [Next: Tietze Extension Theorem →](04-tietze-extension-theorem.md)
