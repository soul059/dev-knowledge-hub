# Chapter 7: Rearrangement of Series

[← Previous: Absolute and Conditional Convergence](06-absolute-and-conditional-convergence.md) | [Back to README](../README.md) | [Next: Unit 4 — Open and Closed Sets →](../04-Topology-of-Real-Line/01-open-and-closed-sets.md)

---

## Overview

The order in which you add terms of an infinite series matters — **if the convergence is conditional**. Riemann's Rearrangement Theorem shows that a conditionally convergent series can be rearranged to converge to *any* prescribed real number, or even to diverge. In contrast, absolutely convergent series are immune to rearrangement. This is one of the most surprising and important results in analysis.

---

## 7.1 Rearrangement — Definition

**Definition 7.1.** A **rearrangement** of $\sum a_n$ is a series $\sum a_{\sigma(n)}$ where $\sigma : \mathbb{N} \to \mathbb{N}$ is a bijection (permutation).

Every term appears exactly once, but in a different order.

```
  Original:      a₁, a₂, a₃, a₄, a₅, a₆, a₇, ...
  Rearrangement: a₂, a₅, a₁, a₇, a₃, a₆, a₄, ...
                  ↑   ↑   ↑   ↑   ↑   ↑   ↑
                 σ(1) σ(2) σ(3) ...
  
  σ is a bijection on ℕ: no term is repeated or lost
```

---

## 7.2 Absolute Convergence is Rearrangement-Invariant

**Theorem 7.2.** If $\sum a_n$ converges absolutely to $S$, then every rearrangement $\sum a_{\sigma(n)}$ also converges absolutely to $S$.

**Proof.** Let $\varepsilon > 0$. Choose $N$ so that $\sum_{n=N+1}^{\infty} |a_n| < \varepsilon$.

Since $\sigma$ is a bijection, choose $M$ large enough that $\{1, 2, \ldots, N\} \subseteq \{\sigma(1), \sigma(2), \ldots, \sigma(M)\}$.

For $m \geq M$:

$$\left|\sum_{n=1}^{m} a_{\sigma(n)} - S\right| = \left|\sum_{n=1}^{m} a_{\sigma(n)} - \sum_{n=1}^{\infty} a_n\right|$$

All terms $a_1, \ldots, a_N$ appear in $\sum_{n=1}^m a_{\sigma(n)}$, so the difference only involves terms $a_k$ with $k > N$:

$$\left|\sum_{n=1}^{m} a_{\sigma(n)} - S\right| \leq \sum_{n=N+1}^{\infty} |a_n| < \varepsilon$$

Hence $\sum a_{\sigma(n)} = S$. The absolute convergence of the rearrangement follows similarly. $\blacksquare$

```
  Why this works (intuition):

  Absolutely convergent: terms decay fast enough that
  reordering cannot accumulate significant error.

  Original:       |a₁| + |a₂| + |a₃| + ... = finite
  Any reordering:  same terms, same total = same finite sum

  ┌──────────────────────────────────────────────┐
  │  ABSOLUTE CONVERGENCE = ORDER DOESN'T MATTER │
  └──────────────────────────────────────────────┘
```

---

## 7.3 Riemann Rearrangement Theorem

**Theorem 7.3 (Riemann, 1854).** If $\sum a_n$ converges **conditionally**, then for any $L \in \mathbb{R}$ (or $L = \pm\infty$), there exists a rearrangement $\sum a_{\sigma(n)}$ that converges to $L$.

**Proof sketch (for finite $L$).** Since $\sum a_n$ converges conditionally:
- $\sum a_n^+ = +\infty$ and $\sum a_n^- = +\infty$ (both parts diverge)
- $a_n \to 0$ (divergence test), so $a_n^+ \to 0$ and $a_n^- \to 0$

**Algorithm to achieve target $L$:**

1. Add positive terms (in their original order) until the partial sum exceeds $L$.
2. Then add negative terms until the partial sum drops below $L$.
3. Repeat.

```
  Rearrangement algorithm targeting L:

  Partial
  sum
    ▲
    │     ╱╲
    │   ╱    ╲
  L ├─╱──────╲──╱╲────╱╲──── → L
    │╱         ╲╱  ╲╱╱  ╲╱
    │
    └──────────────────────────► terms added
         + + + - - + + - + - -

  Step 1: Add positive terms until > L  (always possible: Σa⁺ = ∞)
  Step 2: Add negative terms until < L  (always possible: Σa⁻ = ∞)
  Step 3: Repeat — oscillations shrink since aₙ → 0
```

**Why it converges to $L$:** At each stage, the partial sum overshoots $L$ by at most one term. Since $a_n^+ \to 0$ and $a_n^- \to 0$, the overshoot tends to 0. So the partial sums converge to $L$. $\blacksquare$

---

## 7.4 Worked Example: Rearranging the Alternating Harmonic Series

The alternating harmonic series $\sum (-1)^{n+1}/n = \ln 2$.

**Rearrangement:** Take 2 positive terms, then 1 negative term:

$$\underbrace{1 + \frac{1}{3}}_{\text{2 pos.}} - \underbrace{\frac{1}{2}}_{\text{1 neg.}} + \underbrace{\frac{1}{5} + \frac{1}{7}}_{\text{2 pos.}} - \underbrace{\frac{1}{4}}_{\text{1 neg.}} + \cdots$$

**Claim:** This rearrangement converges to $\frac{3}{2}\ln 2 \neq \ln 2$.

**Derivation:** Group terms:
$$\sum_{k=0}^{\infty}\left(\frac{1}{4k+1} + \frac{1}{4k+3} - \frac{1}{2k+2}\right)$$

$$= \sum_{k=0}^{\infty}\left(\frac{1}{4k+1} + \frac{1}{4k+3} - \frac{1}{4k+2} - \frac{1}{4k+4} + \frac{1}{4k+4} - \frac{1}{2k+2} + \frac{1}{4k+2}\right)$$

After careful algebra, one obtains $\frac{3}{2}\ln 2$.

```
  Original order:   1 - 1/2 + 1/3 - 1/4 + 1/5 - ...  = ln 2

  2 pos, 1 neg:     1 + 1/3 - 1/2 + 1/5 + 1/7 - 1/4  = (3/2) ln 2
                            + ...

  Same terms, different order → different sum!
  
  ┌──────────────────────────────────────────────────┐
  │ This is ONLY possible because the convergence    │
  │ is conditional, not absolute.                    │
  └──────────────────────────────────────────────────┘
```

---

## 7.5 Products of Series (Cauchy Product)

**Definition 7.4.** The **Cauchy product** of $\sum a_n$ and $\sum b_n$ is $\sum c_n$ where:

$$c_n = \sum_{k=0}^{n} a_k b_{n-k}$$

This corresponds to the "diagonal summing" of the product table $a_i \cdot b_j$.

**Theorem 7.5 (Mertens).** If $\sum a_n$ converges absolutely to $A$ and $\sum b_n$ converges to $B$, then the Cauchy product converges to $AB$.

**Theorem 7.6.** If both $\sum a_n$ and $\sum b_n$ converge absolutely, then the Cauchy product converges absolutely to $AB$.

```
  Product table:

           b₀    b₁    b₂    b₃   ...
       ┌─────┬─────┬─────┬─────┬
  a₀   │a₀b₀│a₀b₁│a₀b₂│a₀b₃│ ...
       ├─────┼─────┼─────┼─────┼
  a₁   │a₁b₀│a₁b₁│a₁b₂│     │
       ├─────┼─────┼─────┼─────┼
  a₂   │a₂b₀│a₂b₁│     │     │
       ├─────┼─────┼─────┼─────┼
  a₃   │a₃b₀│     │     │     │
       ├─────┼
    .  │     │
    .  │     │

  Cauchy product: sum along anti-diagonals
  c₀ = a₀b₀
  c₁ = a₀b₁ + a₁b₀
  c₂ = a₀b₂ + a₁b₁ + a₂b₀
  ...
```

---

## 7.6 Unconditional Convergence

**Definition 7.5.** A series $\sum a_n$ is **unconditionally convergent** if every rearrangement converges (to the same sum).

**Theorem 7.7.** In $\mathbb{R}$, $\sum a_n$ converges unconditionally $\iff$ $\sum a_n$ converges absolutely.

```
  ┌───────────────────────────────────────────────┐
  │               EQUIVALENCE IN ℝ                │
  │                                               │
  │  Absolute Convergence ⟺ Unconditional Conv.  │
  │                                               │
  │  (In infinite-dimensional spaces, this        │
  │   equivalence can fail!)                      │
  └───────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Result |
|---------|------------|
| Rearrangement | Bijection $\sigma : \mathbb{N} \to \mathbb{N}$; same terms, different order |
| Absolute conv. & rearrangement | Every rearrangement converges to the same sum |
| Riemann Rearrangement Thm. | Conditional ⟹ can rearrange to any target $L \in \mathbb{R} \cup \{\pm\infty\}$ |
| Algorithm | Add positive terms until $> L$, negative until $< L$, repeat |
| Example | Alt. harmonic: rearranging 2 pos, 1 neg gives $\frac{3}{2}\ln 2$ |
| Cauchy product | $c_n = \sum_{k=0}^n a_k b_{n-k}$; Mertens: works if one is abs. conv. |
| Unconditional conv. | $\iff$ absolute convergence (in $\mathbb{R}$) |

---

## Quick Revision Questions

1. **State** Riemann's Rearrangement Theorem. Why does it require *conditional* convergence?

2. **Prove** that every rearrangement of an absolutely convergent series converges to the same sum.

3. **Explain** the algorithm for rearranging a conditionally convergent series to sum to a target $L$.

4. **Give an example** of a rearrangement of $\sum(-1)^{n+1}/n$ that sums to $+\infty$.

5. **Define** the Cauchy product and state Mertens' Theorem.

6. **True or False:** If $\sum a_n$ converges but not absolutely, then there exists a rearrangement that diverges. Justify.

---

[← Previous: Absolute and Conditional Convergence](06-absolute-and-conditional-convergence.md) | [Back to README](../README.md) | [Next: Unit 4 — Open and Closed Sets →](../04-Topology-of-Real-Line/01-open-and-closed-sets.md)
