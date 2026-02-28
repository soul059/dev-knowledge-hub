# Chapter 3: Ratio and Root Tests

[← Previous: Comparison Tests](02-comparison-tests.md) | [Back to README](../README.md) | [Next: Integral Test →](04-integral-test.md)

---

## Overview

The Ratio and Root Tests use the internal structure of terms (ratios of consecutive terms or $n$th roots) to compare a series to a geometric series. They are especially useful for series involving factorials, exponentials, and powers.

---

## 3.1 The Ratio Test (d'Alembert)

**Theorem 3.1 (Ratio Test).** Let $\sum a_n$ with $a_n \neq 0$ for all $n$, and let:

$$L = \lim_{n \to \infty} \left|\frac{a_{n+1}}{a_n}\right|$$

| $L$ | Conclusion |
|-----|------------|
| $L < 1$ | $\sum a_n$ converges absolutely |
| $L > 1$ | $\sum a_n$ diverges |
| $L = 1$ | **Inconclusive** |

**Proof ($L < 1$).** Choose $r$ with $L < r < 1$. For large enough $n \geq N$: $|a_{n+1}|/|a_n| < r$.

Then $|a_N+k| < r^k |a_N|$, so $\sum_{k=0}^{\infty} |a_{N+k}| \leq |a_N| \sum_{k=0}^{\infty} r^k = |a_N|/(1-r) < \infty$. $\blacksquare$

**Proof ($L > 1$).** For large $n$: $|a_{n+1}| > |a_n|$, so $|a_n| \not\to 0$, hence $\sum a_n$ diverges by the Divergence Test. $\blacksquare$

```
  Ratio Test — Geometric series comparison:

  If |aₙ₊₁/aₙ| → L < 1:

  |aₙ|           The terms eventually decay
    ▲             FASTER than a geometric rⁿ
    │ ●
    │   ●
    │     ●  ← actual |aₙ|
    │       ● ·
    │         ·●· ← geometric rⁿ for comparison
    │            · ·●· ·
    │                  · · ● · ·
    └────────────────────────────────► n

  Series converges by comparison with geometric series.
```

---

## 3.2 The Root Test (Cauchy)

**Theorem 3.2 (Root Test).** Let $\sum a_n$ and:

$$L = \limsup_{n \to \infty} |a_n|^{1/n}$$

| $L$ | Conclusion |
|-----|------------|
| $L < 1$ | $\sum a_n$ converges absolutely |
| $L > 1$ | $\sum a_n$ diverges |
| $L = 1$ | **Inconclusive** |

**Proof ($L < 1$).** Choose $r$ with $L < r < 1$. By definition of limsup, $|a_n|^{1/n} < r$ for all $n \geq N$. So $|a_n| < r^n$ for $n \geq N$, and $\sum |a_n|$ converges by comparison with $\sum r^n$. $\blacksquare$

---

## 3.3 Ratio Test vs Root Test

| Feature | Ratio Test | Root Test |
|---------|-----------|-----------|
| Computes | $\lim |a_{n+1}/a_n|$ | $\limsup |a_n|^{1/n}$ |
| Best for | Factorials, products | $n$-th powers |
| Strength | Sometimes inconclusive when Root is not | Always at least as strong as Ratio |
| Example strength | $a_n = 2^{(-1)^n n}$ — Ratio fails, Root works | |

**Theorem 3.3.** If $\lim |a_{n+1}/a_n| = L$ exists, then $\limsup |a_n|^{1/n} = L$ as well.

The Root Test is **at least as powerful** as the Ratio Test: whenever the Ratio Test gives a conclusion, the Root Test gives the same. But the Root Test can sometimes succeed when the Ratio Test is inconclusive.

---

## 3.4 When $L = 1$: Inconclusive Cases

| Series | $L$ (Ratio) | Converges? |
|--------|:-----------:|:----------:|
| $\sum 1/n$ | $1$ | ❌ |
| $\sum 1/n^2$ | $1$ | ✅ |
| $\sum 1/n^3$ | $1$ | ✅ |

Both convergent and divergent series give $L = 1$, so **neither test can decide**.

---

## 3.5 Worked Examples

### Example 1: $\sum \frac{n!}{n^n}$

$$\left|\frac{a_{n+1}}{a_n}\right| = \frac{(n+1)!}{(n+1)^{n+1}} \cdot \frac{n^n}{n!} = \frac{n^n}{(n+1)^n} = \left(\frac{n}{n+1}\right)^n = \frac{1}{(1+1/n)^n} \to \frac{1}{e} < 1$$

By Ratio Test: **converges**.

### Example 2: $\sum \frac{2^n}{n!}$

$$\left|\frac{a_{n+1}}{a_n}\right| = \frac{2^{n+1}}{(n+1)!} \cdot \frac{n!}{2^n} = \frac{2}{n+1} \to 0 < 1$$

By Ratio Test: **converges**.

### Example 3: $\sum \left(\frac{n}{2n+1}\right)^n$

$$|a_n|^{1/n} = \frac{n}{2n+1} \to \frac{1}{2} < 1$$

By Root Test: **converges**.

### Example 4: $\sum \frac{n^{10}}{3^n}$

$$\left|\frac{a_{n+1}}{a_n}\right| = \frac{(n+1)^{10}}{3^{n+1}} \cdot \frac{3^n}{n^{10}} = \frac{1}{3}\left(\frac{n+1}{n}\right)^{10} \to \frac{1}{3} < 1$$

By Ratio Test: **converges**. (Exponentials dominate polynomials.)

---

## 3.6 Decision Flowchart

```
  Choosing between Ratio and Root:
  ═════════════════════════════════

  Does aₙ involve n! or products?
     │
     ├── YES → Try RATIO TEST first
     │
     └── NO → Does aₙ have the form (f(n))ⁿ?
               │
               ├── YES → Try ROOT TEST
               │
               └── NO → Try RATIO TEST
                         │
                         └── Gives L = 1? → Use other tests
                                              (Comparison, Integral, etc.)
```

---

## Summary Table

| Test | Formula | Converges | Diverges | Inconclusive |
|------|---------|-----------|----------|:------------:|
| Ratio | $L = \lim\|a_{n+1}/a_n\|$ | $L < 1$ | $L > 1$ | $L = 1$ |
| Root | $L = \limsup\|a_n\|^{1/n}$ | $L < 1$ | $L > 1$ | $L = 1$ |

| Key Facts | |
|-----------|---|
| Root ≥ Ratio in power | Root Test works whenever Ratio does, and sometimes when Ratio fails |
| Best for factorials | Ratio Test |
| Best for $n$th powers | Root Test |
| Both fail at $L = 1$ | Use comparison or integral test instead |

---

## Quick Revision Questions

1. **State and prove the Ratio Test** for the case $L < 1$.

2. **Apply the Ratio Test** to $\sum \frac{3^n}{n!}$. What is $L$?

3. **Apply the Root Test** to $\sum \left(\frac{2n+1}{3n+2}\right)^n$.

4. **Give an example** where the Ratio Test is inconclusive but the Root Test succeeds.

5. **True or False:** If the Ratio Test gives $L = 1$, the series might converge or diverge. Give examples of each.

6. **Prove** that if $\lim|a_{n+1}/a_n|$ exists and equals $L$, then $\limsup|a_n|^{1/n} = L$.

---

[← Previous: Comparison Tests](02-comparison-tests.md) | [Back to README](../README.md) | [Next: Integral Test →](04-integral-test.md)
