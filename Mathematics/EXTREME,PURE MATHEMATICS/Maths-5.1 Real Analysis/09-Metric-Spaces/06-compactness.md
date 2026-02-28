# Chapter 6: Compactness in Metric Spaces

[← Previous: Completeness](05-completeness.md) | [Back to README](../README.md)

---

## Overview

Compactness in metric spaces generalizes the Heine–Borel property from $\mathbb{R}^n$. In arbitrary metric spaces, **compact $\neq$ closed + bounded** in general. Instead, compactness is characterized by **sequential compactness** and **total boundedness + completeness**.

---

## 6.1 Definitions

**Definition 6.1 (Open Cover Compactness).** $(K, d)$ is **compact** if every open cover has a finite subcover:

$$K \subseteq \bigcup_{\alpha} U_\alpha \;\Rightarrow\; K \subseteq U_{\alpha_1} \cup \cdots \cup U_{\alpha_n}$$

**Definition 6.1' (Sequential Compactness).** $K$ is **sequentially compact** if every sequence in $K$ has a convergent subsequence (with limit in $K$).

---

## 6.2 Equivalence in Metric Spaces

**Theorem 6.2.** In a metric space, the following are **equivalent**:

1. $K$ is compact (open cover)
2. $K$ is sequentially compact
3. $K$ is **complete** and **totally bounded**

```
  Equivalence of compactness notions (metric spaces):

  ┌──────────────────────────────────────────┐
  │   compact     ⟺   sequentially compact  │
  │       ⟺                                  │
  │   complete + totally bounded              │
  └──────────────────────────────────────────┘

  In general topological spaces, these differ!
  But in metric spaces, all three are the same.
```

---

## 6.3 Total Boundedness

**Definition 6.3.** $A \subseteq X$ is **totally bounded** if:

$$\forall\, \varepsilon > 0,\; \exists\, x_1, \ldots, x_n \in A: \; A \subseteq \bigcup_{k=1}^n B(x_k, \varepsilon)$$

(Finitely many $\varepsilon$-balls cover $A$.)

**Totally bounded $\Rightarrow$ bounded** (but not conversely in general).

**Example of bounded but not totally bounded:** The unit ball in $\ell^\infty$. Take $e_n = (0,\ldots,0,1,0,\ldots)$: $d(e_m, e_n) = 1$ for $m \neq n$, so no finite $1/2$-net exists.

---

## 6.4 Heine–Borel Fails in General

**In $\mathbb{R}^n$:** Compact $\iff$ closed + bounded (Heine–Borel). ✓

**In general metric spaces:** Closed + bounded $\not\Rightarrow$ compact.

**Counterexample.** In $\ell^2$, the closed unit ball $\overline{B}(\mathbf{0}, 1) = \{(x_n) : \sum x_n^2 \leq 1\}$ is closed and bounded but NOT compact. The sequence $e_1, e_2, e_3, \ldots$ has $\|e_m - e_n\| = \sqrt{2}$, so no convergent subsequence.

```
  Heine-Borel: where it works and fails

  ℝⁿ:    compact ⟺ closed + bounded       ✓

  ℓ²:    closed unit ball is closed + bounded
         but NOT compact                     ✗
         (missing: total boundedness)

  General: compact ⟺ complete + totally bounded
```

---

## 6.5 Properties of Compact Sets

**Theorem 6.5.** In metric spaces:

| Property | Statement |
|----------|-----------|
| Compact ⟹ closed + bounded | Always |
| Closed ⊂ compact | Closed subset of compact is compact |
| Continuous image | $f$ continuous, $K$ compact $\Rightarrow$ $f(K)$ compact |
| EVT | $f: K \to \mathbb{R}$ continuous, $K$ compact $\Rightarrow$ $f$ attains max/min |
| Uniform continuity | $f: K \to Y$ continuous, $K$ compact $\Rightarrow$ $f$ uniformly continuous |
| Lebesgue number | Every open cover of compact $K$ has a Lebesgue number $\delta > 0$ |

---

## 6.6 Compactness in $C[a,b]$: Arzelà–Ascoli

**Question.** When is a subset of $(C[a,b], d_\infty)$ compact?

Closed + bounded is NOT enough (infinite-dimensional!). We need an extra condition.

**Definition 6.6.** $\mathcal{F} \subseteq C[a,b]$ is **equicontinuous** if:

$$\forall\, \varepsilon > 0,\; \exists\, \delta > 0: |x - y| < \delta \Rightarrow |f(x) - f(y)| < \varepsilon \;\;\forall\, f \in \mathcal{F}$$

(Same $\delta$ works for ALL functions in $\mathcal{F}$.)

**Theorem 6.6 (Arzelà–Ascoli).** $\mathcal{F} \subseteq C[a,b]$ is compact (in $d_\infty$) $\iff$:
1. $\mathcal{F}$ is **closed**
2. $\mathcal{F}$ is **uniformly bounded**: $\exists\, M$: $\|f\|_\infty \leq M$ for all $f \in \mathcal{F}$
3. $\mathcal{F}$ is **equicontinuous**

```
  Arzelà–Ascoli (compactness in C[a,b]):

  In ℝⁿ:      compact ⟺ closed + bounded
  In C[a,b]:   compact ⟺ closed + bounded + equicontinuous
                                              ↑ extra condition!

  Equicontinuity replaces total boundedness
  in the function space setting.
```

**Why equicontinuity?** It provides the "totally bounded" condition. Without it, functions can oscillate faster and faster, preventing convergent subsequences.

---

## 6.7 Summary: Compactness Across Spaces

| Space | Compact $\iff$ |
|-------|----------------|
| $\mathbb{R}^n$ | Closed + bounded (Heine–Borel) |
| General metric | Complete + totally bounded |
| General metric | Sequentially compact |
| $C[a,b]$ | Closed + uniformly bounded + equicontinuous (Arzelà–Ascoli) |

---

## Summary Table

| Concept | Details |
|---------|---------|
| Compact | Every open cover has finite subcover |
| Seq. compact | Every sequence has convergent subsequence |
| Totally bounded | Finite $\varepsilon$-nets for all $\varepsilon$ |
| Equivalence | Compact ⟺ seq. compact ⟺ complete + tot. bounded |
| Heine–Borel | Only in $\mathbb{R}^n$: compact ⟺ closed + bounded |
| Arzelà–Ascoli | Compact in $C[a,b]$ ⟺ closed + bounded + equicontinuous |

---

## Quick Revision Questions

1. **Define** compactness and sequential compactness. Prove their equivalence in metric spaces.

2. **Define** total boundedness. Show bounded $\not\Rightarrow$ totally bounded with a counterexample.

3. **Show** Heine–Borel fails in $\ell^2$: give a closed bounded set that is not compact.

4. **State** the Arzelà–Ascoli theorem. Why is equicontinuity needed?

5. **Prove** continuous image of a compact set is compact.

6. **Prove** a continuous function on a compact metric space is uniformly continuous.

---

[← Previous: Completeness](05-completeness.md) | [Back to README](../README.md)
