# Chapter 4: Supremum and Infimum

[← Previous: Completeness Axiom](03-completeness-axiom.md) | [Back to README](../README.md) | [Next: Archimedean Property →](05-archimedean-property.md)

---

## Overview

The supremum and infimum are the most important concepts in real analysis. They allow us to precisely describe the "boundary" of a set even when the set has no maximum or minimum. Mastering the ε-characterization of sup and inf is essential for nearly every proof in this course.

---

## 4.1 Definitions

**Definition 4.1 (Supremum).** Let $S \subseteq \mathbb{R}$, $S \neq \emptyset$, bounded above. The **supremum** of $S$, written $\sup S$, is the number $\alpha \in \mathbb{R}$ satisfying:
1. $s \leq \alpha$ for all $s \in S$ (upper bound)
2. If $M < \alpha$, then $M$ is NOT an upper bound of $S$ (least such)

**Definition 4.2 (Infimum).** Let $S \subseteq \mathbb{R}$, $S \neq \emptyset$, bounded below. The **infimum** of $S$, written $\inf S$, is the number $\beta \in \mathbb{R}$ satisfying:
1. $\beta \leq s$ for all $s \in S$ (lower bound)
2. If $m > \beta$, then $m$ is NOT a lower bound of $S$ (greatest such)

---

## 4.2 Supremum vs Maximum

```
  KEY DISTINCTION:
  ═══════════════

  Maximum: greatest element IN the set
  Supremum: least upper bound (may or may not be in the set)

  Example 1: S = [0, 1]
  ●══════════════●
  0              1
  max S = 1 ✅ (attained)
  sup S = 1 ✅

  Example 2: S = [0, 1)
  ●══════════════○
  0              1
  max S = ??? ❌ (does not exist!)
  sup S = 1 ✅ (exists but NOT in S)

  Example 3: S = {1/n : n ∈ ℕ} = {1, 1/2, 1/3, ...}
  ●  ● ● ●●●●●●○
  0            1
  inf S = 0 (not in S)
  min S does not exist
  sup S = max S = 1 (in S)
```

---

## 4.3 The ε-Characterization (THE Most Important Tool)

**Theorem 4.1 (ε-characterization of supremum).** $\alpha = \sup S$ if and only if:
1. $s \leq \alpha$ for all $s \in S$
2. For every $\varepsilon > 0$, there exists $s_\varepsilon \in S$ such that $\alpha - \varepsilon < s_\varepsilon$

**Theorem 4.2 (ε-characterization of infimum).** $\beta = \inf S$ if and only if:
1. $\beta \leq s$ for all $s \in S$
2. For every $\varepsilon > 0$, there exists $s_\varepsilon \in S$ such that $s_\varepsilon < \beta + \varepsilon$

```
  ε-Characterization of sup S = α:

                  ε           ε
              ◄───────►   ◄───────►
  ────────────┼───────────┼───────────────
           α - ε         α         beyond
                    ▲
                    │
            There MUST be some
            element s ∈ S here
            (between α-ε and α)

  No matter how small ε is, you can always find
  an element of S within ε of α from below.
```

**Proof of Theorem 4.1.**

(⟹) Suppose $\alpha = \sup S$. Condition 1 holds by definition. For condition 2: let $\varepsilon > 0$. Then $\alpha - \varepsilon < \alpha$, so $\alpha - \varepsilon$ is NOT an upper bound (since $\alpha$ is the least upper bound). Thus $\exists\, s_\varepsilon \in S$ with $s_\varepsilon > \alpha - \varepsilon$.

(⟸) Condition 1 says $\alpha$ is an upper bound. For minimality: if $M < \alpha$, set $\varepsilon = \alpha - M > 0$. By condition 2, $\exists\, s \in S$ with $s > \alpha - \varepsilon = M$. So $M$ is not an upper bound. $\blacksquare$

---

## 4.4 Worked Examples

### Example 1: $S = \{1 - 1/n : n \in \mathbb{N}\} = \{0, 1/2, 2/3, 3/4, \ldots\}$

**Claim:** $\sup S = 1$.

**Proof:**
1. Upper bound: $1 - 1/n < 1$ for all $n \in \mathbb{N}$. ✓
2. ε-characterization: Given $\varepsilon > 0$, choose $n$ with $n > 1/\varepsilon$ (Archimedean property). Then $1 - 1/n > 1 - \varepsilon$. ✓

**Claim:** $\inf S = 0$.

**Proof:** $0 = 1 - 1/1 \in S$, and $1 - 1/n \geq 0$ for all $n$. So $0$ is the minimum, hence also the infimum. ✓

### Example 2: $S = \{x \in \mathbb{R} : x^2 < 2\} = (-\sqrt{2}, \sqrt{2})$

**Claim:** $\sup S = \sqrt{2}$.

1. Upper bound: If $x^2 < 2$, then $|x| < \sqrt{2}$, so $x < \sqrt{2}$. ✓
2. ε-characterization: Given $\varepsilon > 0$, take $s = \sqrt{2} - \varepsilon/2$ (for small enough $\varepsilon$). Then $s \in S$ and $s > \sqrt{2} - \varepsilon$. ✓

### Example 3: Supremum of a Union

**Theorem 4.3.** If $A, B \subseteq \mathbb{R}$ are nonempty and bounded above, then:
$$\sup(A \cup B) = \max(\sup A, \sup B)$$

**Proof.** Let $\alpha = \sup A$, $\beta = \sup B$, $\gamma = \max(\alpha, \beta)$.
- For all $s \in A \cup B$: $s \leq \alpha \leq \gamma$ or $s \leq \beta \leq \gamma$. So $\gamma$ is an upper bound.
- $\gamma = \alpha$ or $\gamma = \beta$. In either case, $\gamma \in \{\alpha, \beta\}$ is itself a sup of some set, so no smaller upper bound exists. $\blacksquare$

---

## 4.5 Arithmetic with Suprema

**Theorem 4.4.** Let $A, B \subseteq \mathbb{R}$ be nonempty and bounded above. Define:
$$A + B = \{a + b : a \in A,\; b \in B\}$$

Then: $\sup(A + B) = \sup A + \sup B$.

**Proof.** Let $\alpha = \sup A$, $\beta = \sup B$.

*Upper bound:* For all $a + b \in A + B$: $a + b \leq \alpha + \beta$.

*Least upper bound:* Let $\varepsilon > 0$. Choose $a \in A$ with $a > \alpha - \varepsilon/2$ and $b \in B$ with $b > \beta - \varepsilon/2$. Then $a + b > \alpha + \beta - \varepsilon$. By the ε-characterization, $\sup(A + B) = \alpha + \beta$. $\blacksquare$

**Warning:** For products, $\sup(A \cdot B) \neq \sup A \cdot \sup B$ in general (signs matter!).

---

## 4.6 Decision Flowchart: Finding Sup and Inf

```
  Given a set S ⊆ ℝ:
  ═══════════════════

  Step 1: Is S bounded above?
     │
     ├── NO  → sup S = +∞ (does not exist as real number)
     │
     └── YES → Step 2: Guess a candidate α for sup S
                  │
                  ├── Step 3: Verify α is an upper bound
                  │     (show s ≤ α for ALL s ∈ S)
                  │
                  └── Step 4: Verify α is LEAST
                        (use ε-characterization: for any
                         ε > 0, find s ∈ S with s > α - ε)

  Common strategies for Step 2:
  ┌────────────────────────────────────────────┐
  │ • Compute a few elements and look for      │
  │   a pattern approaching some value         │
  │ • Solve for boundary: what value can       │
  │   elements get arbitrarily close to?       │
  │ • Try the limit of a sequence in S         │
  └────────────────────────────────────────────┘
```

---

## 4.7 Extended Real Numbers (Convention)

For convenience, we extend: $\sup \emptyset = -\infty$ and $\inf \emptyset = +\infty$.

For unbounded sets: $\sup S = +\infty$ if $S$ is not bounded above, and $\inf S = -\infty$ if not bounded below.

---

## 4.8 Real-World Application

- **Optimization**: The supremum of a set of feasible objective values is the optimal value (even if not attained)
- **Probability**: The essential supremum defines the "almost everywhere" maximum
- **Signal processing**: Peak signal values are suprema of signal amplitude sets
- **Economics**: Utility maximization often involves suprema when optima are not attained

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Supremum | Least upper bound — exists by completeness |
| Infimum | Greatest lower bound — exists by GLB property |
| Maximum vs Supremum | max must be in the set; sup need not be |
| ε-characterization of sup | $\forall \varepsilon > 0,\; \exists\, s \in S:\; s > \sup S - \varepsilon$ |
| ε-characterization of inf | $\forall \varepsilon > 0,\; \exists\, s \in S:\; s < \inf S + \varepsilon$ |
| $\sup(A \cup B)$ | $= \max(\sup A, \sup B)$ |
| $\sup(A + B)$ | $= \sup A + \sup B$ |
| sup attained ⟹ max exists | If $\sup S \in S$, then $\max S = \sup S$ |

---

## Quick Revision Questions

1. **Define supremum** and explain how it differs from maximum. Give an example where sup exists but max does not.

2. **State and prove the ε-characterization** of the supremum.

3. **Find $\sup S$ and $\inf S$** for $S = \{(-1)^n / n : n \in \mathbb{N}\} = \{-1, 1/2, -1/3, 1/4, \ldots\}$. Are they attained?

4. **Prove that $\sup(A + B) = \sup A + \sup B$** for nonempty bounded sets $A, B \subseteq \mathbb{R}$.

5. **True or False:** If $\sup S = \alpha$ and $s < \alpha$ for all $s \in S$, then $\alpha \notin S$. Explain.

6. **Let $S = \{x \in \mathbb{R} : 0 < x < 1, x \in \mathbb{Q}\}$.** Find $\sup S$ and $\inf S$. Is either attained?

---

[← Previous: Completeness Axiom](03-completeness-axiom.md) | [Back to README](../README.md) | [Next: Archimedean Property →](05-archimedean-property.md)
