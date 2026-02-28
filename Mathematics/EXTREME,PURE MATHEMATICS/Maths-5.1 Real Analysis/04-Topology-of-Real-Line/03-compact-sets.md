# Chapter 3: Compact Sets

[← Previous: Limit Points and Closure](02-limit-points-and-closure.md) | [Back to README](../README.md) | [Next: Heine–Borel Theorem →](04-heine-borel-theorem.md)

---

## Overview

Compactness is one of the most powerful concepts in analysis. A compact set behaves like a "finite" set in many ways: continuous functions on compact sets attain maxima, sequences have convergent subsequences, and open covers have finite subcovers. In $\mathbb{R}$, compactness = closed + bounded.

---

## 3.1 Open Covers

**Definition 3.1.** An **open cover** of $A \subseteq \mathbb{R}$ is a collection $\{O_\alpha\}_{\alpha \in I}$ of open sets whose union contains $A$:

$$A \subseteq \bigcup_{\alpha \in I} O_\alpha$$

A **subcover** is a sub-collection that still covers $A$. A **finite subcover** uses finitely many sets.

```
  Open cover of [0, 1]:

  O₁ = (-0.1, 0.4)    ←────────→
  O₂ = (0.3, 0.7)          ←────────→
  O₃ = (0.6, 1.1)               ←────────→

  ══════[════════════════════]══════
       0                    1

  {O₁, O₂, O₃} covers [0,1] — and it's already finite!
```

---

## 3.2 Definition of Compactness

**Definition 3.2.** A set $K \subseteq \mathbb{R}$ is **compact** if every open cover of $K$ has a finite subcover.

$$\forall\; \text{open cover } \{O_\alpha\} \text{ of } K, \;\exists\; O_{\alpha_1}, \ldots, O_{\alpha_N} \text{ with } K \subseteq \bigcup_{k=1}^N O_{\alpha_k}$$

**Key:** We must check *every possible* open cover, not just specific ones.

```
  Compact: "You can't hide from finitely many open sets"

  Non-compact example: (0, 1) with cover Oₙ = (1/n, 1)
  
  O₂ = (1/2, 1)       ──────────→
  O₃ = (1/3, 1)    ─────────────→
  O₄ = (1/4, 1)  ───────────────→
    ...              → getting wider
  
  ═══(════════════════)═══
     0                1

  No finite subcover works! Points near 0 always escape.
  So (0, 1) is NOT compact.
```

---

## 3.3 Examples

| Set | Compact? | Reason |
|-----|:--------:|--------|
| $[a, b]$ | ✅ | Heine–Borel (closed + bounded) |
| $(a, b)$ | ❌ | Not closed; cover $\{(1/n, b-1/n)\}$ has no finite subcover |
| $\{x_1, \ldots, x_N\}$ | ✅ | Finite set; one open set per point suffices |
| $\mathbb{R}$ | ❌ | Not bounded; $\{(-n, n)\}$ has no finite subcover |
| $[0, \infty)$ | ❌ | Not bounded |
| $\{1/n : n \in \mathbb{N}\}$ | ❌ | Not closed ($0$ is a limit point not in the set) |
| $\{0\} \cup \{1/n\}$ | ✅ | Closed + bounded |
| Cantor set | ✅ | Closed + bounded |

---

## 3.4 Compact Sets are Closed and Bounded

**Theorem 3.3.** If $K$ is compact, then $K$ is closed and bounded.

**Proof (bounded).** $\{(-n, n) : n \in \mathbb{N}\}$ covers $K$. Extract finite subcover: $K \subseteq (-N, N)$. So $K$ is bounded. $\blacksquare$

**Proof (closed).** We show $K^c$ is open. Fix $y \notin K$. For each $x \in K$, choose disjoint open intervals: $U_x \ni x$ and $V_x \ni y$ with $U_x \cap V_x = \emptyset$ (possible since $x \neq y$; take radius $|x-y|/2$).

$\{U_x : x \in K\}$ covers $K$. By compactness, extract finite subcover $U_{x_1}, \ldots, U_{x_N}$.

Let $V = V_{x_1} \cap \cdots \cap V_{x_N}$. This is a finite intersection of open sets containing $y$, so $V$ is open and $y \in V$. Also $V \cap K \subseteq V \cap \bigcup U_{x_k} = \emptyset$. So $V \subseteq K^c$, proving $y$ is interior to $K^c$. $\blacksquare$

```
  Proving K^c is open:

  Fix y ∉ K:
              K
      ┌──[●──●──●──●──●]──┐
      │   x₁ x₂ x₃ x₄ x₅ │    y ●
      └───────────────────┘

  For each xₖ ∈ K:
  ──(──Uxₖ──)────────(──Vxₖ──)──
      xₖ                 y

  Finite subcover: U_{x₁},...,U_{xₙ} covers K
  V = ∩Vxₖ is an open nhbd of y missing K entirely
```

---

## 3.5 Sequential Compactness

**Definition 3.4.** $K$ is **sequentially compact** if every sequence in $K$ has a subsequence converging to a point in $K$.

**Theorem 3.5.** In $\mathbb{R}$ (and any metric space): $K$ is compact $\iff$ $K$ is sequentially compact.

**Proof sketch ($\Leftarrow$).** (Sequentially compact $\Rightarrow$ compact) By contradiction. Suppose open cover $\{O_\alpha\}$ has no finite subcover. Bisect $K$ repeatedly: at each step, one half has no finite subcover. Get nested closed intervals $I_n$ with $|I_n| \to 0$; pick $x_n \in I_n \cap K$. A subsequence converges to some $x \in K$. Then $x \in O_\beta$ for some $\beta$, and $O_\beta$ contains all but finitely many $x_n$, giving a contradiction.

---

## 3.6 Properties of Compact Sets

**Theorem 3.6.**
1. Closed subsets of compact sets are compact
2. If $K$ is compact and $F \subseteq K$ is closed, then $F$ is compact
3. Finite unions of compact sets are compact
4. Arbitrary intersections of compact sets are compact (closed subset of a compact set)
5. If $K_1 \supseteq K_2 \supseteq \cdots$ are non-empty compact sets, then $\bigcap K_n \neq \emptyset$ (Nested Compact Set Property)

**Theorem 3.7 (Nested Compact Set Property).** If $K_1 \supseteq K_2 \supseteq \cdots$ are non-empty compact sets, then $\bigcap_{n=1}^\infty K_n \neq \emptyset$.

**Proof.** Pick $x_n \in K_n$. Then $(x_n) \subseteq K_1$ (compact), so $\exists$ subsequence $x_{n_k} \to x \in K_1$. For each $m$, $x_{n_k} \in K_m$ for $k$ large (since $n_k \geq m$), and $K_m$ is closed, so $x \in K_m$. Thus $x \in \bigcap K_n$. $\blacksquare$

---

## 3.7 Compactness and Continuity (Preview)

Compact sets are crucial because:
- Continuous images of compact sets are compact
- Continuous functions on compact sets are bounded and attain bounds (Extreme Value Theorem)
- Continuous functions on compact sets are uniformly continuous

These results appear in Unit 5.

---

## Summary Table

| Concept | Key Result |
|---------|------------|
| Open cover | Collection of open sets containing $A$ |
| Compact | Every open cover has a finite subcover |
| In $\mathbb{R}$: compact | $\iff$ closed + bounded (Heine–Borel) |
| Sequential compact | Every sequence has convergent subsequence in $K$ |
| Compact = seq. compact | ✅ in $\mathbb{R}$ (and metric spaces) |
| Closed $\subseteq$ compact | $\Rightarrow$ compact |
| Nested compact sets | Intersection is non-empty |

---

## Quick Revision Questions

1. **Define** compactness using open covers. Why must we check *every* open cover?

2. **Show** that $(0, 1)$ is not compact by exhibiting an open cover with no finite subcover.

3. **Prove** that compact sets are closed and bounded.

4. **State** the equivalence of compactness and sequential compactness in $\mathbb{R}$.

5. **Prove** the Nested Compact Set Property.

6. **True or False:** A countable subset of $\mathbb{R}$ is never compact. Justify.

---

[← Previous: Limit Points and Closure](02-limit-points-and-closure.md) | [Back to README](../README.md) | [Next: Heine–Borel Theorem →](04-heine-borel-theorem.md)
