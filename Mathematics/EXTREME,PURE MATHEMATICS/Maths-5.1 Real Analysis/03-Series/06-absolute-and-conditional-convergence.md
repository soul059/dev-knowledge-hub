# Chapter 6: Absolute and Conditional Convergence

[← Previous: Alternating Series](05-alternating-series.md) | [Back to README](../README.md) | [Next: Rearrangement of Series →](07-rearrangement-of-series.md)

---

## Overview

There are two "strengths" of convergence for series: **absolute** (where $\sum |a_n|$ converges) and **conditional** (where $\sum a_n$ converges but $\sum |a_n|$ diverges). Absolute convergence is the "better" kind — it implies convergence and allows rearrangement, while conditional convergence is fragile.

---

## 6.1 Definitions

**Definition 6.1.** A series $\sum a_n$:
- **converges absolutely** if $\sum |a_n|$ converges
- **converges conditionally** if $\sum a_n$ converges but $\sum |a_n|$ diverges

```
  Classification of Series:

                    Σaₙ
                   ╱    ╲
              Converges  Diverges
              ╱      ╲
         Absolutely  Conditionally
         (Σ|aₙ| conv.) (Σ|aₙ| div.)

  ┌──────────────────────────────────────────────┐
  │ Absolute convergence ⟹ Convergence          │
  │ Convergence ⇏ Absolute convergence           │
  └──────────────────────────────────────────────┘
```

---

## 6.2 Absolute Convergence Implies Convergence

**Theorem 6.2.** If $\sum |a_n|$ converges, then $\sum a_n$ converges.

**Proof.** By the Cauchy criterion. Let $\varepsilon > 0$. Since $\sum |a_n|$ converges, $\exists\, N$ with:

$$\sum_{k=n+1}^{m} |a_k| < \varepsilon \quad \text{for } m > n \geq N$$

By the triangle inequality:

$$\left|\sum_{k=n+1}^{m} a_k\right| \leq \sum_{k=n+1}^{m} |a_k| < \varepsilon$$

So the partial sums of $\sum a_n$ are Cauchy, hence convergent. $\blacksquare$

---

## 6.3 Examples

| Series | $\sum \|a_n\|$ | $\sum a_n$ | Type |
|--------|:----------:|:---------:|------|
| $\sum (-1)^n/n^2$ | $\sum 1/n^2$ conv. | Conv. | **Absolutely convergent** |
| $\sum (-1)^n/n$ | $\sum 1/n$ div. | Conv. (AST) | **Conditionally convergent** |
| $\sum (-1)^n$ | $\sum 1$ div. | Div. | **Divergent** |
| $\sum \sin(n)/n^2$ | $\leq \sum 1/n^2$ conv. | Conv. | **Absolutely convergent** |
| $\sum (-1)^n/\sqrt{n}$ | $\sum 1/\sqrt{n}$ div. | Conv. (AST) | **Conditionally convergent** |

---

## 6.4 Why Absolute Convergence is "Better"

| Property | Absolutely convergent | Conditionally convergent |
|----------|:--------------------:|:------------------------:|
| Implies convergence | ✅ | (is convergent by def.) |
| Rearrangement invariant | ✅ (same sum) | ❌ (sum changes!) |
| Product of series converges | ✅ | Not guaranteed |
| All convergence tests apply | ✅ | Limited tests |

The key difference is **rearrangement** — explored in detail in Chapter 7.

---

## 6.5 Positive and Negative Parts

For any series $\sum a_n$, define:

$$a_n^+ = \max(a_n, 0) = \frac{|a_n| + a_n}{2}, \qquad a_n^- = \max(-a_n, 0) = \frac{|a_n| - a_n}{2}$$

Then $a_n = a_n^+ - a_n^-$ and $|a_n| = a_n^+ + a_n^-$.

**Theorem 6.3.** If $\sum a_n$ converges absolutely:
- Both $\sum a_n^+$ and $\sum a_n^-$ converge
- $\sum a_n = \sum a_n^+ - \sum a_n^-$

If $\sum a_n$ converges conditionally:
- Both $\sum a_n^+ = +\infty$ and $\sum a_n^- = +\infty$

```
  Conditional convergence: positive and negative parts both diverge!

  Partial sums:
    ▲
    │      ╱╲      ╱╲      ╱╲
    │    ╱    ╲  ╱    ╲  ╱    ╲  → S
    │  ╱       ╲╱      ╲╲      → ───
    │╱                    ╲╱
    └────────────────────────────► n

  The positive terms push the sum UP (+∞ worth)
  The negative terms pull the sum DOWN (-∞ worth)
  But they CANCEL to give a finite sum S.
  
  This cancellation is why rearranging is dangerous!
```

---

## 6.6 Tests for Absolute Convergence

All the convergence tests from previous chapters test for **absolute convergence** when applied to $\sum |a_n|$:

| Test | Applied to $\sum \|a_n\|$ gives... |
|------|----------------------------------|
| Comparison | Absolute convergence |
| Ratio Test | Absolute convergence (when $L < 1$) |
| Root Test | Absolute convergence (when $L < 1$) |
| Integral Test | Absolute convergence |

**The Alternating Series Test is the main tool for conditional convergence.**

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Absolute convergence | $\sum \|a_n\|$ converges |
| Conditional convergence | $\sum a_n$ conv. but $\sum \|a_n\|$ div. |
| Absolute ⟹ convergence | Triangle inequality + Cauchy criterion |
| Converse fails | $\sum(-1)^n/n$: convergent but not absolutely |
| Positive/negative parts | Both diverge for conditional convergence |
| Rearrangement safe? | Only for absolute convergence |
| Tests | Ratio, Root, Comparison test absolute; AST tests conditional |

---

## Quick Revision Questions

1. **Define absolute and conditional convergence.** Give an example of each.

2. **Prove** that absolute convergence implies convergence.

3. **Classify** each: (a) $\sum (-1)^n/n^3$ (b) $\sum (-1)^n/\sqrt{n}$ (c) $\sum \sin(n)/2^n$.

4. **Prove:** If $\sum a_n$ is conditionally convergent, then $\sum a_n^+ = +\infty$ and $\sum a_n^- = +\infty$.

5. **Why is absolute convergence considered "safer"** than conditional convergence?

6. **True or False:** The Ratio Test can detect conditional convergence. Justify.

---

[← Previous: Alternating Series](05-alternating-series.md) | [Back to README](../README.md) | [Next: Rearrangement of Series →](07-rearrangement-of-series.md)
