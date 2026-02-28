# Chapter 1: Convergence Definition (ε-N)

[← Previous: Density of Rationals](../01-Real-Number-System/06-density-of-rationals-and-irrationals.md) | [Back to README](../README.md) | [Next: Limit Theorems →](02-limit-theorems.md)

---

## Overview

The concept of convergence is the heartbeat of real analysis. A sequence "converges" to a limit $L$ if its terms get arbitrarily close to $L$ and stay close. The ε-N definition makes this precise — it is the single most important definition in this entire course.

---

## 1.1 Sequences — Basic Definitions

**Definition.** A **sequence** of real numbers is a function $a : \mathbb{N} \to \mathbb{R}$. We write $a_n = a(n)$ and denote the sequence by $(a_n)_{n=1}^{\infty}$ or simply $(a_n)$.

**Examples:**
| Sequence | First few terms | Formula |
|----------|----------------|---------|
| $(1/n)$ | $1, 1/2, 1/3, 1/4, \ldots$ | $a_n = 1/n$ |
| $((-1)^n)$ | $-1, 1, -1, 1, \ldots$ | $a_n = (-1)^n$ |
| $(n^2)$ | $1, 4, 9, 16, \ldots$ | $a_n = n^2$ |
| $(1 + 1/n)^n$ | $2, 2.25, 2.37, \ldots$ | Approaches $e$ |

---

## 1.2 The ε-N Definition of Convergence

**Definition 1.1 (Convergence).** A sequence $(a_n)$ **converges to** $L \in \mathbb{R}$ if:

$$\forall\, \varepsilon > 0,\; \exists\, N \in \mathbb{N} \;\text{such that}\; \forall\, n \geq N:\; |a_n - L| < \varepsilon$$

We write $\lim_{n \to \infty} a_n = L$ or $a_n \to L$.

A sequence that does not converge is called **divergent**.

```
  THE ε-N DEFINITION — Visual Interpretation:

  Value
    ▲
    │         ε-band around L
    │    ┌─────────────────────────────────────────
  L+ε ───┤· · · · · · · · · · · · · · · · · · · ·
    │    │    ●                    ●
    │    │       ●         ●          ●    ●   ●
  L ─────┤──────────●──●──────●──────────●──●──────
    │    │    ●          ●        ●
    │    │ ●       ●                   ●
  L-ε ───┤· · · · · · · · · · · · · · · · · · · ·
    │    └─────────────────────────────────────────
    │    ▲
    │    N    From index N onward, ALL terms
    │         are within ε of L
    └──────────────────────────────────────────► n
         1   2   3   4   5   6   7   8   9  ...
```

---

## 1.3 Unpacking the Definition

The logical structure has **three quantifiers**, and their ORDER matters:

```
  ∀ ε > 0         "For EVERY positive tolerance..."
    │
    └──► ∃ N ∈ ℕ       "...there EXISTS a threshold..."
            │
            └──► ∀ n ≥ N     "...such that ALL terms past N..."
                    │
                    └──► |aₙ - L| < ε    "...are within ε of L"

  CRITICAL: The order is  ∀ε → ∃N → ∀n
  
  • ε is given FIRST (by an "adversary")
  • N is chosen SECOND (by us, depending on ε)
  • The condition must hold for ALL n ≥ N
```

### What Convergence Does NOT Mean

| Misconception | Correction |
|--------------|------------|
| "Terms get closer to $L$ at every step" | NO — terms can oscillate, as long as they're eventually within ε |
| "Eventually $a_n = L$" | NO — terms need only be close, not equal |
| "$N$ is the same for all ε" | NO — smaller ε generally requires larger $N$ |
| "Only finitely many terms outside any ε-band" | YES — this is correct! |

---

## 1.4 Worked Examples

### Example 1: $a_n = 1/n \to 0$

**Proof.** Let $\varepsilon > 0$ be given. By the Archimedean property, choose $N \in \mathbb{N}$ with $N > 1/\varepsilon$.

For $n \geq N$:
$$|a_n - 0| = \frac{1}{n} \leq \frac{1}{N} < \varepsilon$$
$\blacksquare$

```
  Finding N for aₙ = 1/n:

  ε = 0.1  →  Need 1/n < 0.1  →  n > 10   →  N = 11
  ε = 0.01 →  Need 1/n < 0.01 →  n > 100  →  N = 101
  ε = 0.001 → Need 1/n < 0.001 → n > 1000 →  N = 1001

  Smaller ε requires larger N — this is typical.
```

### Example 2: $a_n = (3n + 1)/(2n + 5) \to 3/2$

**Proof.** Let $\varepsilon > 0$.

$$|a_n - 3/2| = \left|\frac{3n+1}{2n+5} - \frac{3}{2}\right| = \left|\frac{2(3n+1) - 3(2n+5)}{2(2n+5)}\right| = \left|\frac{-13}{2(2n+5)}\right| = \frac{13}{2(2n+5)}$$

We need $\frac{13}{2(2n+5)} < \varepsilon$, i.e., $n > \frac{13 - 10\varepsilon}{4\varepsilon}$.

Choose $N > \frac{13}{4\varepsilon}$ (a simpler bound since $\frac{13-10\varepsilon}{4\varepsilon} < \frac{13}{4\varepsilon}$).

For $n \geq N$: $|a_n - 3/2| = \frac{13}{2(2n+5)} < \frac{13}{4n} \leq \frac{13}{4N} < \varepsilon$. $\blacksquare$

### Example 3: $a_n = (-1)^n$ diverges

**Proof.** Suppose $a_n \to L$ for some $L \in \mathbb{R}$. Take $\varepsilon = 1/2$.

Then $\exists\, N$ such that $\forall\, n \geq N$: $|a_n - L| < 1/2$.

For even $n \geq N$: $|1 - L| < 1/2$, so $L > 1/2$.
For odd $n \geq N$: $|-1 - L| < 1/2$, so $L < -1/2$.

But $L > 1/2$ and $L < -1/2$ is impossible. Contradiction. $\blacksquare$

---

## 1.5 Uniqueness of Limits

**Theorem 1.2.** If $a_n \to L$ and $a_n \to M$, then $L = M$.

**Proof.** Let $\varepsilon > 0$. Choose $N_1$ with $|a_n - L| < \varepsilon/2$ for $n \geq N_1$, and $N_2$ with $|a_n - M| < \varepsilon/2$ for $n \geq N_2$. Let $N = \max(N_1, N_2)$.

For $n \geq N$:
$$|L - M| = |L - a_n + a_n - M| \leq |a_n - L| + |a_n - M| < \frac{\varepsilon}{2} + \frac{\varepsilon}{2} = \varepsilon$$

Since $|L - M| < \varepsilon$ for all $\varepsilon > 0$, we get $L = M$. $\blacksquare$

---

## 1.6 Convergent Sequences are Bounded

**Theorem 1.3.** If $(a_n)$ converges, then $(a_n)$ is bounded.

**Proof.** Let $a_n \to L$. With $\varepsilon = 1$, choose $N$ such that $|a_n - L| < 1$ for $n \geq N$. Then $|a_n| < |L| + 1$ for $n \geq N$.

Let $M = \max(|a_1|, |a_2|, \ldots, |a_{N-1}|, |L| + 1)$. Then $|a_n| \leq M$ for all $n$. $\blacksquare$

**Warning:** The converse is false! $a_n = (-1)^n$ is bounded but divergent.

---

## 1.7 Proof Strategy Template

```
  Template for proving aₙ → L:
  ═════════════════════════════

  Proof. Let ε > 0 be given.            ◄── Always start here

  [Scratch work — not part of proof:
   Solve |aₙ - L| < ε for n.
   Get n > f(ε) for some expression f.]

  Choose N ∈ ℕ with N > f(ε).           ◄── State your choice of N
                                             (use Archimedean property
                                              if needed)

  For n ≥ N:                             ◄── Verify the condition
    |aₙ - L| = ...                           (chain of inequalities)
              ≤ ...
              < ε.                       ◄── Must end with < ε

  ∴ aₙ → L.  □
```

---

## 1.8 The Negation of Convergence

$(a_n)$ does **not** converge to $L$ if:
$$\exists\, \varepsilon_0 > 0 \;\text{such that}\; \forall\, N \in \mathbb{N},\; \exists\, n \geq N:\; |a_n - L| \geq \varepsilon_0$$

$(a_n)$ **diverges** (converges to nothing) if:
$$\forall\, L \in \mathbb{R},\; \exists\, \varepsilon_0 > 0 \;\text{such that}\; \forall\, N \in \mathbb{N},\; \exists\, n \geq N:\; |a_n - L| \geq \varepsilon_0$$

```
  Negation logic:

  Converges to L:     ∀ε > 0,  ∃N,  ∀n ≥ N:  |aₙ - L| < ε
                         │       │      │          │
  Negate each:        ∃ε₀ > 0, ∀N,  ∃n ≥ N:  |aₙ - L| ≥ ε₀
  (flip ∀↔∃, flip <)
```

---

## 1.9 Real-World Application

- **Numerical methods**: Iterative algorithms (Newton-Raphson, gradient descent) produce sequences that should converge to a solution. The ε-N definition tells us precisely what "convergence" means.
- **Signal processing**: A discrete signal converges if its samples approach a steady-state value.
- **Economics**: Time series of prices, GDP, etc. are sequences; convergence models equilibrium.

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Sequence | Function $a : \mathbb{N} \to \mathbb{R}$ |
| Convergence ($a_n \to L$) | $\forall \varepsilon > 0,\;\exists N:\;\forall n \geq N,\; \|a_n - L\| < \varepsilon$ |
| Limit is unique | Triangle inequality argument |
| Convergent ⟹ bounded | Finite initial segment + tail bound |
| Bounded ⇏ convergent | Counterexample: $(-1)^n$ |
| Divergence | Negation of the convergence definition |
| Proof strategy | Scratch work to find $N$, then verify |

---

## Quick Revision Questions

1. **State the ε-N definition of convergence.** Explain the role of each quantifier.

2. **Prove from the definition** that $\frac{2n}{n+3} \to 2$.

3. **Prove that the limit of a convergent sequence is unique.** Where is the triangle inequality used?

4. **Write the negation** of "$a_n \to L$." Use it to prove $(-1)^n$ does not converge.

5. **True or False:** If $|a_n - L| < 1/n$ for all $n$, then $a_n \to L$. Prove or disprove.

6. **Give an example** of a bounded sequence that diverges, and an unbounded sequence. Does every unbounded sequence diverge?

---

[← Previous: Density of Rationals](../01-Real-Number-System/06-density-of-rationals-and-irrationals.md) | [Back to README](../README.md) | [Next: Limit Theorems →](02-limit-theorems.md)
