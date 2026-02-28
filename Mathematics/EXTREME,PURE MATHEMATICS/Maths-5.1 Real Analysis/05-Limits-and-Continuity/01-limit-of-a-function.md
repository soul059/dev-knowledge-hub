# Chapter 1: Limit of a Function

[← Previous: Unit 4 — Perfect Sets](../04-Topology-of-Real-Line/06-perfect-sets.md) | [Back to README](../README.md) | [Next: Properties of Limits →](02-properties-of-limits.md)

---

## Overview

The $\varepsilon$-$\delta$ definition of a function limit is the foundation of continuity, differentiation, and integration. We define $\lim_{x \to c} f(x) = L$ and develop both the $\varepsilon$-$\delta$ and sequential characterizations.

---

## 1.1 The ε-δ Definition

**Definition 1.1.** Let $f: A \to \mathbb{R}$ and $c$ be a limit point of $A$. We say:

$$\lim_{x \to c} f(x) = L$$

if $\forall\, \varepsilon > 0$, $\exists\, \delta > 0$ such that:

$$0 < |x - c| < \delta \;\text{ and }\; x \in A \quad\Rightarrow\quad |f(x) - L| < \varepsilon$$

```
  The ε-δ picture:

  y ▲
    │     ┌─────────── L + ε
    │     │  ╱ ●╲
    │ ····│╱·····╲····· L
    │     ╱       ╲
    │   ╱└─────────── L - ε
    │  ╱
    └──┼──┼──┼──┼──────► x
       c-δ  c  c+δ

  For ANY ε-band around L,
  there EXISTS a δ-band around c
  such that f maps (c-δ, c+δ) ∩ A \ {c}
  into (L-ε, L+ε).
```

**Critical notes:**
- $c$ must be a limit point of $A$ (so points near $c$ exist in $A$)
- We require $0 < |x - c|$ — the value $f(c)$ is irrelevant (may not even be defined)
- $\delta$ depends on $\varepsilon$ (and potentially on $c$)

---

## 1.2 Quantifier Structure

$$\lim_{x \to c} f(x) = L \quad\iff\quad \forall\, \varepsilon > 0,\; \exists\, \delta > 0,\; \forall\, x \in A: \; 0 < |x-c| < \delta \Rightarrow |f(x) - L| < \varepsilon$$

**Negation:**

$$\lim_{x \to c} f(x) \neq L \quad\iff\quad \exists\, \varepsilon_0 > 0,\; \forall\, \delta > 0,\; \exists\, x \in A: \; 0 < |x-c| < \delta \;\text{ and }\; |f(x) - L| \geq \varepsilon_0$$

```
  Quantifier structure comparison:

  Sequence limit:       Function limit:
  ∀ε > 0               ∀ε > 0
    ∃N ∈ ℕ               ∃δ > 0
      ∀n ≥ N               ∀x: 0 < |x-c| < δ
        |aₙ - L| < ε        |f(x) - L| < ε

  N (discrete) ↔ δ (continuous)
```

---

## 1.3 Worked Examples

**Example 1.** $\lim_{x \to 3} (4x - 5) = 7$.

*Proof.* Let $\varepsilon > 0$. We need $|4x - 5 - 7| < \varepsilon$, i.e., $4|x - 3| < \varepsilon$. Choose $\delta = \varepsilon/4$.

If $0 < |x - 3| < \delta$, then $|(4x-5) - 7| = 4|x-3| < 4\delta = \varepsilon$. $\blacksquare$

**Example 2.** $\lim_{x \to 2} x^2 = 4$.

*Proof.* $|x^2 - 4| = |x-2||x+2|$. We need to bound $|x+2|$. If $|x-2| < 1$, then $|x| < 3$, so $|x+2| \leq |x| + 2 < 5$.

Choose $\delta = \min(1, \varepsilon/5)$. If $0 < |x-2| < \delta$:

$$|x^2 - 4| = |x-2||x+2| < \frac{\varepsilon}{5} \cdot 5 = \varepsilon. \quad\blacksquare$$

**Example 3.** $\lim_{x \to 0} \sin(1/x)$ does not exist.

*Proof.* Take $x_n = 1/(n\pi)$ and $y_n = 1/((2n + 1/2)\pi)$. Both $x_n, y_n \to 0$, but $\sin(1/x_n) = \sin(n\pi) = 0$ while $\sin(1/y_n) = \sin((2n+1/2)\pi) = 1$. Different subsequential limits, so the limit doesn't exist. $\blacksquare$

---

## 1.4 Sequential Characterization

**Theorem 1.2 (Sequential Criterion).** Let $c$ be a limit point of $A$. Then:

$$\lim_{x \to c} f(x) = L \quad\iff\quad \text{for every sequence } (x_n) \subseteq A \setminus \{c\} \text{ with } x_n \to c,\; f(x_n) \to L$$

**Proof ($\Rightarrow$).** Given $\varepsilon$-$\delta$ definition and $x_n \to c$ with $x_n \neq c$. Choose $N$ with $|x_n - c| < \delta$ for $n \geq N$. Then $|f(x_n) - L| < \varepsilon$.

**Proof ($\Leftarrow$).** Contrapositive. If the $\varepsilon$-$\delta$ definition fails, $\exists\, \varepsilon_0 > 0$ such that $\forall\, \delta_n = 1/n$, $\exists\, x_n$ with $0 < |x_n - c| < 1/n$ and $|f(x_n) - L| \geq \varepsilon_0$. Then $x_n \to c$ but $f(x_n) \not\to L$. $\blacksquare$

**Application:** The Sequential Criterion is the main tool for proving limits *don't* exist — find two sequences approaching $c$ with different limits of $f$.

---

## 1.5 One-Sided Limits

**Definition 1.3.**

$$\lim_{x \to c^+} f(x) = L \quad\iff\quad \forall\, \varepsilon > 0,\; \exists\, \delta > 0: \; 0 < x - c < \delta \Rightarrow |f(x) - L| < \varepsilon$$

$$\lim_{x \to c^-} f(x) = L \quad\iff\quad \forall\, \varepsilon > 0,\; \exists\, \delta > 0: \; 0 < c - x < \delta \Rightarrow |f(x) - L| < \varepsilon$$

**Theorem 1.4.** $\lim_{x \to c} f(x) = L \;\iff\; \lim_{x \to c^+} f(x) = L = \lim_{x \to c^-} f(x)$.

```
  One-sided limits:

  f(x) ▲
       │      ●  ← f(c) (irrelevant)
    L₊ │- - - ╱         Right limit
       │    ╱
       │  ╱
    L₋ │─╲──────         Left limit
       │   ╲
       └────┼────────► x
            c

  If L₊ ≠ L₋, the (two-sided) limit does not exist.
  Example: f(x) = |x|/x at x = 0: L₋ = -1, L₊ = +1.
```

---

## 1.6 Infinite Limits and Limits at Infinity

**Definition 1.5.** $\lim_{x \to c} f(x) = +\infty$ means:

$$\forall\, M > 0,\; \exists\, \delta > 0: \; 0 < |x - c| < \delta \Rightarrow f(x) > M$$

**Definition 1.6.** $\lim_{x \to \infty} f(x) = L$ means:

$$\forall\, \varepsilon > 0,\; \exists\, M > 0: \; x > M \Rightarrow |f(x) - L| < \varepsilon$$

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| $\lim_{x \to c} f(x) = L$ | $\forall\, \varepsilon > 0$, $\exists\, \delta > 0$: $0 < \|x-c\| < \delta \Rightarrow \|f(x)-L\| < \varepsilon$ |
| Negation | $\exists\, \varepsilon_0$, $\forall\, \delta$, $\exists\, x$: $0 < \|x-c\| < \delta$ but $\|f(x)-L\| \geq \varepsilon_0$ |
| Sequential criterion | Limit $= L$ iff $f(x_n) \to L$ for all $x_n \to c$, $x_n \neq c$ |
| $f(c)$ irrelevant | Limit depends only on behavior near $c$, not at $c$ |
| One-sided limits | Left and right limits; two-sided exists iff both agree |
| Proving limit | Find $\delta$ in terms of $\varepsilon$ (often use scratch work) |
| Disproving limit | Sequential criterion with two sequences |

---

## Quick Revision Questions

1. **State** the $\varepsilon$-$\delta$ definition of $\lim_{x \to c} f(x) = L$.

2. **Prove** from the definition that $\lim_{x \to 1} (3x + 2) = 5$.

3. **Prove** $\lim_{x \to 4} \sqrt{x} = 2$ using $\varepsilon$-$\delta$.

4. **Show** that $\lim_{x \to 0} \sin(1/x)$ does not exist using the Sequential Criterion.

5. **State and prove** the Sequential Criterion for function limits.

6. **True or False:** If $\lim_{x \to c^+} f(x) = L$ and $\lim_{x \to c^-} f(x) = L$, then $f(c) = L$. Justify.

---

[← Previous: Unit 4 — Perfect Sets](../04-Topology-of-Real-Line/06-perfect-sets.md) | [Back to README](../README.md) | [Next: Properties of Limits →](02-properties-of-limits.md)
