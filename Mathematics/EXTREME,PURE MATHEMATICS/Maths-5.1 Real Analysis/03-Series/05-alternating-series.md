# Chapter 5: Alternating Series

[← Previous: Integral Test](04-integral-test.md) | [Back to README](../README.md) | [Next: Absolute and Conditional Convergence →](06-absolute-and-conditional-convergence.md)

---

## Overview

An alternating series has terms that switch sign: positive, negative, positive, negative, ... The Alternating Series Test (Leibniz test) gives a simple sufficient condition for convergence and provides an excellent error bound.

---

## 5.1 Definition

**Definition 5.1.** An **alternating series** is a series of the form:

$$\sum_{n=1}^{\infty} (-1)^{n+1} b_n = b_1 - b_2 + b_3 - b_4 + \cdots$$

where $b_n > 0$ for all $n$.

---

## 5.2 The Alternating Series Test (Leibniz)

**Theorem 5.2 (Alternating Series Test).** If $(b_n)$ satisfies:
1. $b_n \geq b_{n+1}$ for all $n$ (decreasing)
2. $\lim_{n \to \infty} b_n = 0$

Then $\sum_{n=1}^{\infty} (-1)^{n+1} b_n$ converges.

**Proof.** Consider the even partial sums:

$$S_{2n} = (b_1 - b_2) + (b_3 - b_4) + \cdots + (b_{2n-1} - b_{2n})$$

Each parenthesized pair $\geq 0$ (since $b_k \geq b_{k+1}$), so $(S_{2n})$ is **increasing**.

Also: $S_{2n} = b_1 - (b_2 - b_3) - (b_4 - b_5) - \cdots - (b_{2n-2} - b_{2n-1}) - b_{2n} \leq b_1$.

So $(S_{2n})$ is increasing and bounded above by $b_1$. By MCT, $S_{2n} \to S$ for some $S$.

Now $S_{2n+1} = S_{2n} + b_{2n+1} \to S + 0 = S$.

Since both even and odd partial sums converge to $S$, the full sequence $S_n \to S$. $\blacksquare$

```
  Alternating Series — Partial Sums Oscillate and Converge:

  Value
    ▲
    │  S₁ ●
    │         S₃ ●
    │              S₅ ●
  S ─┤─────────────────●──●──●──●── limit
    │           S₄ ●
    │      S₂ ●
    │
    └──────────────────────────────────► n

  Even sums (S₂ₙ): increase ↑ toward S
  Odd sums (S₂ₙ₊₁): decrease ↓ toward S
  
  S₂ₙ ≤ S ≤ S₂ₙ₊₁  (the sum is "trapped")
```

---

## 5.3 Alternating Series Remainder Estimate

**Theorem 5.3 (Remainder Estimate).** For an alternating series satisfying the conditions of the AST:

$$|S - S_N| \leq b_{N+1}$$

The error is bounded by the **first omitted term**.

**Proof.** $S$ lies between $S_N$ and $S_{N+1}$, so $|S - S_N| \leq |S_{N+1} - S_N| = b_{N+1}$. $\blacksquare$

This is an **extremely useful** estimate — much tighter than what most other tests provide.

---

## 5.4 Examples

### Example 1: Alternating Harmonic Series

$$\sum_{n=1}^{\infty} \frac{(-1)^{n+1}}{n} = 1 - \frac{1}{2} + \frac{1}{3} - \frac{1}{4} + \cdots = \ln 2$$

**Check AST:** $b_n = 1/n$, decreasing ✓, $b_n \to 0$ ✓. So the series converges.

Note: $\sum 1/n$ diverges, but $\sum (-1)^{n+1}/n$ converges! This is **conditional convergence** (Chapter 6).

### Example 2: $\sum_{n=1}^{\infty} \frac{(-1)^n}{\sqrt{n}}$

$b_n = 1/\sqrt{n}$: decreasing ✓, $\to 0$ ✓. **Converges** by AST.

### Example 3: Does $\sum (-1)^n \frac{n}{n+1}$ converge?

$b_n = n/(n+1) \to 1 \neq 0$. Condition 2 fails. By the Divergence Test, the series **diverges**.

---

## 5.5 Conditions Are Necessary

| Condition violated | Example | Result |
|-------------------|---------|--------|
| $b_n \not\to 0$ | $\sum (-1)^n$ | Diverges (Divergence Test) |
| $b_n$ not decreasing | $b_n = 1/n$ for odd $n$, $b_n = 1/n^2$ for even $n$ | May still converge, but AST doesn't apply |

The AST is **sufficient but not necessary** — some alternating series converge even without the decreasing condition.

---

## 5.6 Comparison Table

| Series | $b_n$ | Decreasing? | $b_n \to 0$? | AST? | Verdict |
|--------|-------|:-----------:|:------------:|:----:|---------|
| $\sum (-1)^n/n$ | $1/n$ | ✅ | ✅ | ✅ | Converges |
| $\sum (-1)^n/n^2$ | $1/n^2$ | ✅ | ✅ | ✅ | Converges |
| $\sum (-1)^n/\sqrt{n}$ | $1/\sqrt{n}$ | ✅ | ✅ | ✅ | Converges |
| $\sum (-1)^n \cdot n/(n+1)$ | $n/(n+1)$ | ❌ | ❌ | ❌ | Diverges |
| $\sum (-1)^n/\ln n$ | $1/\ln n$ | ✅ | ✅ | ✅ | Converges |

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Alternating series | $\sum(-1)^{n+1} b_n$ with $b_n > 0$ |
| AST conditions | $b_n$ decreasing and $b_n \to 0$ |
| AST conclusion | The series converges |
| Proof idea | Even/odd partial sums are monotone and bounded |
| Remainder estimate | $\|S - S_N\| \leq b_{N+1}$ (first omitted term) |
| Key example | $\sum(-1)^{n+1}/n = \ln 2$ |
| AST is sufficient, not necessary | Some alternating series converge without the conditions |

---

## Quick Revision Questions

1. **State and prove the Alternating Series Test.**

2. **Show that $\sum_{n=2}^{\infty} \frac{(-1)^n}{\ln n}$ converges.** How many terms are needed to approximate the sum within $0.1$?

3. **Prove the Remainder Estimate** $|S - S_N| \leq b_{N+1}$.

4. **Does $\sum_{n=1}^{\infty} (-1)^n \frac{n^2}{n^2 + 1}$ converge?** Why or why not?

5. **True or False:** The alternating harmonic series converges absolutely.

6. **Explain** why the even and odd partial sums of an alternating series approach the same limit.

---

[← Previous: Integral Test](04-integral-test.md) | [Back to README](../README.md) | [Next: Absolute and Conditional Convergence →](06-absolute-and-conditional-convergence.md)
