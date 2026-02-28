# Chapter 4: Heine–Borel Theorem

[← Previous: Compact Sets](03-compact-sets.md) | [Back to README](../README.md) | [Next: Connected Sets →](05-connected-sets.md)

---

## Overview

The **Heine–Borel Theorem** is the cornerstone characterization of compactness in $\mathbb{R}^n$: a subset is compact if and only if it is **closed and bounded**. This converts an abstract topological property (every open cover has a finite subcover) into a concrete, easily checkable condition.

---

## 4.1 Statement

**Theorem 4.1 (Heine–Borel).** For $K \subseteq \mathbb{R}$:

$$K \text{ is compact} \quad\iff\quad K \text{ is closed and bounded}$$

```
  Heine–Borel: The Grand Equivalence in ℝ

  ┌───────────────────────────────┐
  │  Every open cover             │
  │  has a finite subcover        │
  │         ⟺                    │
  │  K is closed AND bounded      │
  │         ⟺                    │
  │  Every sequence in K has a    │
  │  subsequence converging in K  │
  └───────────────────────────────┘

  Three equivalent views of the same property!
```

The direction **compact $\Rightarrow$ closed and bounded** was proved in Chapter 3. The important (and harder) direction is **closed and bounded $\Rightarrow$ compact**.

---

## 4.2 Proof: Closed and Bounded ⟹ Compact

**Proof.** Let $K$ be closed and bounded. Since $K$ is bounded, $K \subseteq [-M, M]$ for some $M > 0$.

**Step 1:** Prove $[a, b]$ is compact (Lemma).

**Step 2:** Closed subset of a compact set is compact. Since $K$ is closed and $K \subseteq [-M, M]$, and $[-M, M]$ is compact, $K$ is compact.

**Lemma:** $[a, b]$ is compact.

**Proof of Lemma (by contradiction + LUB).** Let $\{O_\alpha\}$ be an open cover of $[a, b]$. Define:

$$S = \{x \in [a, b] : [a, x] \text{ can be covered by finitely many } O_\alpha\}$$

**Claim:** $\sup S = b$ and $b \in S$.

- $S \neq \emptyset$: $a \in O_\beta$ for some $\beta$, and since $O_\beta$ is open, $[a, a + \delta] \subseteq O_\beta$ for some $\delta > 0$. So $a + \delta \in S$.
- $S$ is bounded above by $b$, so $s = \sup S$ exists and $s \leq b$.

**$s \in S$:** $s \in O_\gamma$ for some $\gamma$ (since $\{O_\alpha\}$ covers $[a, b]$). Since $O_\gamma$ is open, $\exists\, \varepsilon > 0$ with $(s - \varepsilon, s + \varepsilon) \subseteq O_\gamma$. By definition of sup, $\exists\, x_0 \in S$ with $x_0 > s - \varepsilon$. So $[a, x_0]$ has a finite subcover, and adding $O_\gamma$ covers $[a, s]$. Thus $s \in S$.

**$s = b$:** If $s < b$, then $O_\gamma$ also covers $[a, s + \varepsilon']$ for small $\varepsilon' > 0$, so $s + \varepsilon' \in S$, contradicting $s = \sup S$.

So $[a, b]$ has a finite subcover. $\blacksquare$

```
  Proof visualization:

  a                           s = sup S                    b
  [═══════════════════════════|═══════════]
  ◄──── covered by finitely  ─►
         many Oα's

  s sits in some Oγ (open):
       (s-ε ─────── s ─────── s+ε)  ⊆ Oγ

  Some x₀ in (s-ε, s) is in S, so [a, x₀] has finite subcover.
  Adding Oγ covers [a, s].  → s ∈ S.
  If s < b, we can push further into Oγ → contradiction.
  So s = b.  ∎
```

---

## 4.3 The Full Equivalence Chain

In $\mathbb{R}$, the following are equivalent for $K \subseteq \mathbb{R}$:

| # | Property |
|---|----------|
| 1 | $K$ is compact (every open cover has a finite subcover) |
| 2 | $K$ is sequentially compact (every sequence has convergent subseq. in $K$) |
| 3 | $K$ is closed and bounded |
| 4 | $K$ is complete and totally bounded |

**Proof cycle:** (1) $\Rightarrow$ (3) [Ch. 3], (3) $\Rightarrow$ (1) [this chapter], (1) $\iff$ (2) [Ch. 3/BW], (3) $\iff$ (4) [metric space theory].

```
  Equivalence chain:

       Compact ────────► Closed & Bounded
         ▲                    │
         │                    ▼
  Sequentially ◄──── Heine–Borel proves
    Compact           this direction
```

---

## 4.4 Why Heine–Borel Fails Outside ℝⁿ

**Warning:** Heine–Borel (closed + bounded $\iff$ compact) is specific to $\mathbb{R}^n$!

In general metric spaces, closed + bounded does NOT imply compact.

**Counterexample:** In $\ell^\infty$ (bounded sequences with sup norm), the closed unit ball $\{x : \|x\| \leq 1\}$ is closed and bounded but NOT compact. The standard basis vectors $e_1, e_2, e_3, \ldots$ form a sequence with no convergent subsequence ($\|e_n - e_m\| = 1$ for $n \neq m$).

---

## 4.5 Applications of Heine–Borel

**Application 1: Quick compactness checks.**
- Is $[0, 1] \cup [2, 3]$ compact? Yes — closed and bounded. ✅
- Is $[0, \infty)$ compact? No — not bounded. ❌
- Is $(0, 1)$ compact? No — not closed. ❌

**Application 2: The Cantor set is compact.**
The Cantor set $C$ is: (a) bounded (contained in $[0,1]$); (b) closed (intersection of closed sets $C_n$). By Heine–Borel, $C$ is compact.

**Application 3: Extreme Value Theorem setup.**
If $K$ is compact and $f: K \to \mathbb{R}$ is continuous, then $f(K)$ is compact (closed + bounded), so $f$ attains its maximum and minimum.

---

## 4.6 Lebesgue Number Lemma

**Lemma 4.2 (Lebesgue).** If $\{O_\alpha\}$ is an open cover of a compact set $K$, then $\exists\, \delta > 0$ (called the **Lebesgue number**) such that every subset of $K$ with diameter $< \delta$ is contained in some single $O_\alpha$.

This lemma is useful for proving uniform continuity on compact sets.

---

## Summary Table

| Concept | Key Result |
|---------|------------|
| Heine–Borel | Compact $\iff$ closed + bounded (in $\mathbb{R}$) |
| Key proof technique | LUB property to show $[a,b]$ is compact |
| Closed $\subseteq$ compact | $\Rightarrow$ compact |
| Fails outside $\mathbb{R}^n$ | Closed + bounded $\not\Rightarrow$ compact in general |
| Quick test | Check closed AND bounded — that's all you need in $\mathbb{R}$ |
| Lebesgue number | Uniform "mesh size" for open covers of compact sets |

---

## Quick Revision Questions

1. **State** the Heine–Borel Theorem. In which spaces does it hold?

2. **Prove** that $[a, b]$ is compact using the LUB property.

3. **Use Heine–Borel** to determine which are compact: (a) $\{1/n : n \in \mathbb{N}\}$ (b) $[0,1] \cap \mathbb{Q}$ (c) Cantor set.

4. **Why** does closed + bounded fail to imply compact in general metric spaces? Give a counterexample.

5. **Prove:** A closed subset of a compact set is compact.

6. **State** the Lebesgue Number Lemma and explain its significance for uniform continuity.

---

[← Previous: Compact Sets](03-compact-sets.md) | [Back to README](../README.md) | [Next: Connected Sets →](05-connected-sets.md)
