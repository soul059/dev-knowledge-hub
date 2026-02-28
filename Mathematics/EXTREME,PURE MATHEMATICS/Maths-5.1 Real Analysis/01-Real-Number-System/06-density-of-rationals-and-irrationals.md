# Chapter 6: Density of Rationals and Irrationals

[← Previous: Archimedean Property](05-archimedean-property.md) | [Back to README](../README.md) | [Next: Unit 2 — Convergence Definition →](../02-Sequences/01-convergence-definition.md)

---

## Overview

Despite ℚ having "gaps" (it's incomplete), rational numbers are densely sprinkled throughout ℝ — between any two real numbers, there's always a rational. Remarkably, the same is true of irrationals. This chapter proves these density results and explores cardinality.

---

## 6.1 Density of ℚ in ℝ

**Theorem 6.1 (Density of Rationals).** For every $a, b \in \mathbb{R}$ with $a < b$, there exists $r \in \mathbb{Q}$ with $a < r < b$.

**Proof.**

**Step 1.** By the Archimedean property, choose $n \in \mathbb{N}$ with $n > \frac{1}{b - a}$, i.e., $\frac{1}{n} < b - a$.

**Step 2.** By the Archimedean property again, the set $\{m \in \mathbb{Z} : m > na\}$ is nonempty. Let $m$ be its minimum (by well-ordering applied to a shift).

Then $m - 1 \leq na < m$, so $a < m/n$.

**Step 3.** We show $m/n < b$. From $m \leq na + 1$:
$$\frac{m}{n} \leq a + \frac{1}{n} < a + (b - a) = b$$

So $a < m/n < b$, and $r = m/n \in \mathbb{Q}$. $\blacksquare$

```
  Proof Strategy — Finding a rational between a and b:

  ◄─────────────────────────────────────────────────────►
         a                                    b
         ├────────────────────────────────────┤
                      b - a

  Step 1: Choose n large enough that 1/n < b - a
          (so the "grid spacing" is fine enough)

  ◄──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──►
    0  1/n 2/n 3/n ...     m/n              grid of
                            ▲                spacing 1/n
                            │
  Step 2: Pick m = smallest integer with m/n > a

  Step 3: Since 1/n < b-a, we get m/n < b ✓
```

---

## 6.2 Density of Irrationals in ℝ

**Theorem 6.2 (Density of Irrationals).** For every $a, b \in \mathbb{R}$ with $a < b$, there exists an irrational number $t$ with $a < t < b$.

**Proof.** Since $a < b$, we have $a - \sqrt{2} < b - \sqrt{2}$. By density of rationals, there exists $r \in \mathbb{Q}$ with:
$$a - \sqrt{2} < r < b - \sqrt{2}$$
Let $t = r + \sqrt{2}$. Then $a < t < b$.

**Claim:** $t$ is irrational. If $t$ were rational, then $\sqrt{2} = t - r$ would be rational (difference of rationals), contradiction. $\blacksquare$

---

## 6.3 Visualizing Density

```
  Between ANY two real numbers, no matter how close:

  ◄──────────────────────────────────────────────────►
              a                    b
              │                    │
       ┌──────┼────────────────────┼──────┐
       │      ●  ○  ●  ○  ●  ○  ● ○    │
       │      ●  ○  ●  ○  ●  ○  ● ○    │
       │      Q  I  Q  I  Q  I  Q I ... │
       └────────────────────────────────┘
              ● = rational    ○ = irrational

  Both rationals AND irrationals are DENSE in ℝ:
  • Between any two reals, there are infinitely many rationals
  • Between any two reals, there are infinitely many irrationals
  • Between any two rationals, there's an irrational
  • Between any two irrationals, there's a rational
```

---

## 6.4 Countability and Cardinality

Even though both ℚ and ℝ\ℚ are dense, they have **very different sizes**.

### Theorem 6.3: ℚ is Countable

A set $S$ is **countable** if there exists a bijection $f : \mathbb{N} \to S$.

**Proof sketch.** List all fractions $m/n$ in a grid and count diagonally:

```
  Cantor's Diagonal Enumeration of ℚ⁺:

  n\m│  1     2     3     4     5   ...
  ───┼────────────────────────────────
   1 │ 1/1 → 2/1   3/1 → 4/1   5/1
     │   ↙       ↗   ↙       ↗
   2 │ 1/2   2/2   3/2   4/2   5/2
     │ ↓   ↗   ↙       ↗
   3 │ 1/3   2/3   3/3   4/3   5/3
     │   ↙       ↗
   4 │ 1/4   2/4   3/4   4/4   5/4
     │ ...
   5 │ 1/5   2/5   3/5   4/5   5/5

  Order: 1/1, 2/1, 1/2, 1/3, 2/2, 3/1, 4/1, 3/2, 2/3, 1/4, ...
  (skip duplicates like 2/2 = 1/1)
```

### Theorem 6.4: ℝ is Uncountable (Cantor's Diagonal Argument)

**Proof.** Suppose ℝ is countable, say $\mathbb{R} = \{x_1, x_2, x_3, \ldots\}$. Write each in decimal:

```
  x₁ = 0. d₁₁  d₁₂  d₁₃  d₁₄  ...
  x₂ = 0. d₂₁  d₂₂  d₂₃  d₂₄  ...
  x₃ = 0. d₃₁  d₃₂  d₃₃  d₃₄  ...
  x₄ = 0. d₄₁  d₄₂  d₄₃  d₄₄  ...
  ...        ↘      ↘      ↘      ↘

  Construct y = 0. b₁ b₂ b₃ b₄ ...
  where bₖ ≠ dₖₖ (and bₖ ∉ {0, 9} to avoid ambiguity)

  Then y ≠ xₖ for every k (they differ in the k-th digit)
  But y ∈ [0,1] ⊆ ℝ — CONTRADICTION!
```

### Corollary 6.5: The irrationals ℝ\ℚ are uncountable.

**Proof.** $\mathbb{R} = \mathbb{Q} \cup (\mathbb{R} \setminus \mathbb{Q})$. If $\mathbb{R} \setminus \mathbb{Q}$ were countable, then $\mathbb{R}$ would be a union of two countable sets, hence countable — contradiction. $\blacksquare$

---

## 6.5 Cardinality Summary

```
  SIZE COMPARISON:
  ═══════════════

  ℕ ──── countably infinite ────┬── "small infinity" (ℵ₀)
  ℤ ── countable ───────────────┤
  ℚ ── countable ───────────────┘
                                     STRICT GAP
  ℝ\ℚ ── uncountable ──────────┬── "large infinity" (𝔠 = 2^ℵ₀)
  ℝ  ── uncountable ───────────┘

  Key paradox: ℚ is DENSE in ℝ but is "small" (countable)
               ℝ\ℚ is also DENSE but is "large" (uncountable)
               
  "Most" real numbers are IRRATIONAL!
```

---

## 6.6 Implications for Analysis

| Result | Consequence |
|--------|-------------|
| Density of ℚ | Every real number is a limit of rationals |
| Density of ℝ\ℚ | Every real number is a limit of irrationals |
| ℚ countable | We can enumerate rationals in proofs |
| ℝ uncountable | "More" irrationals than rationals |
| Both dense | Neither ℚ nor ℝ\ℚ contains an interval |

**Theorem 6.6.** ℚ contains no interval $(a, b)$ with $a < b$.

**Proof.** Any interval $(a, b)$ is uncountable (it bijects with ℝ via $x \mapsto \tan(\pi(x - a)/(b - a) - \pi/2)$). Since ℚ is countable, ℚ cannot contain $(a, b)$. $\blacksquare$

---

## 6.7 Algebraic vs Transcendental Numbers

| Type | Definition | Countable? | Examples |
|------|-----------|-----------|----------|
| Algebraic | Root of a polynomial with integer coefficients | ✅ Yes (countable) | $\sqrt{2}, \sqrt[3]{5}, \frac{3}{7}, i$ |
| Transcendental | NOT algebraic | ❌ No (uncountable) | $\pi, e, 2^{\sqrt{2}}$ |

Since algebraic numbers are countable and ℝ is uncountable, **"most" real numbers are transcendental** — yet proving any specific number is transcendental is usually very hard!

---

## 6.8 Real-World Application

- **Numerical computation**: We approximate irrationals (like $\pi$) by rationals (like 355/113) — density guarantees arbitrary precision is possible
- **Measure theory**: ℚ has measure zero in ℝ; "almost all" numbers are irrational
- **Cryptography**: The density of primes in ℕ (prime number theorem) mirrors density themes
- **Signal processing**: Rational frequencies produce periodic signals; irrational frequencies produce quasiperiodic ones

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Density of ℚ | Between any two reals, there's a rational |
| Density of ℝ\ℚ | Between any two reals, there's an irrational |
| Proof technique for ℚ density | Archimedean property to choose grid spacing |
| Proof technique for ℝ\ℚ density | Shift by $\sqrt{2}$ and use ℚ density |
| ℚ is countable | Cantor's diagonal enumeration |
| ℝ is uncountable | Cantor's diagonal argument |
| ℝ\ℚ is uncountable | Complement argument |
| Algebraic numbers | Countable; roots of integer polynomials |
| Transcendental numbers | Uncountable; "most" reals are transcendental |

---

## Quick Revision Questions

1. **Prove the density of ℚ in ℝ.** Identify exactly where the Archimedean property is used.

2. **Prove the density of irrationals in ℝ.** Why does the "$r + \sqrt{2}$" trick work?

3. **Explain Cantor's diagonal argument.** Why does it prove ℝ is uncountable?

4. **Is the set of irrationals in $(0, 1)$ countable or uncountable?** Prove your answer.

5. **True or False:** Between any two irrational numbers, there exists a rational number. Justify.

6. **Why can ℚ be dense in ℝ yet have "measure zero"?** (Informal explanation is fine.)

---

[← Previous: Archimedean Property](05-archimedean-property.md) | [Back to README](../README.md) | [Next: Unit 2 — Convergence Definition →](../02-Sequences/01-convergence-definition.md)
