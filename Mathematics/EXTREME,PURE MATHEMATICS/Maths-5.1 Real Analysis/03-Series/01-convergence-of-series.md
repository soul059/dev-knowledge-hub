# Chapter 1: Convergence of Series

[← Previous: Completeness](../02-Sequences/07-completeness.md) | [Back to README](../README.md) | [Next: Comparison Tests →](02-comparison-tests.md)

---

## Overview

A **series** is an infinite sum $\sum_{n=1}^{\infty} a_n$. The fundamental question is: does this sum make sense? We define convergence of series via the sequence of partial sums, reducing the study of series to the study of sequences.

---

## 1.1 Definitions

**Definition 1.1.** Given a sequence $(a_n)$, the **series** $\sum_{n=1}^{\infty} a_n$ is defined as the limit of the **partial sums**:

$$S_N = \sum_{n=1}^{N} a_n = a_1 + a_2 + \cdots + a_N$$

The series **converges** if the sequence $(S_N)$ converges. If $S_N \to S$, we write $\sum_{n=1}^{\infty} a_n = S$.

If $(S_N)$ diverges, the series **diverges**.

```
  Series convergence = convergence of partial sums:

  a₁, a₂, a₃, a₄, ...    (terms)
   │
   ▼
  S₁ = a₁
  S₂ = a₁ + a₂
  S₃ = a₁ + a₂ + a₃       (partial sums)
  S₄ = a₁ + a₂ + a₃ + a₄
  ...
   │
   ▼
  Does (Sₙ) converge?
  YES → series converges to lim Sₙ
  NO  → series diverges
```

---

## 1.2 The Geometric Series

**Theorem 1.2.** The geometric series $\sum_{n=0}^{\infty} r^n$ converges if and only if $|r| < 1$, and:

$$\sum_{n=0}^{\infty} r^n = \frac{1}{1-r} \quad \text{for } |r| < 1$$

**Proof.** $S_N = 1 + r + r^2 + \cdots + r^N = \frac{1 - r^{N+1}}{1 - r}$ (for $r \neq 1$).

If $|r| < 1$: $r^{N+1} \to 0$, so $S_N \to \frac{1}{1-r}$.

If $|r| \geq 1$: $r^{N+1}$ does not converge to 0, so $S_N$ diverges. $\blacksquare$

| $r$ | Convergent? | Sum |
|-----|:-----------:|-----|
| $1/2$ | ✅ | $2$ |
| $1/3$ | ✅ | $3/2$ |
| $-1/2$ | ✅ | $2/3$ |
| $0.99$ | ✅ | $100$ |
| $1$ | ❌ | Diverges |
| $-1$ | ❌ | Diverges |
| $2$ | ❌ | Diverges |

---

## 1.3 The Harmonic Series

**Theorem 1.3.** The harmonic series $\sum_{n=1}^{\infty} \frac{1}{n}$ diverges.

**Proof (Condensation argument / Oresme).** Group terms:

$$\sum_{n=1}^{\infty} \frac{1}{n} = 1 + \frac{1}{2} + \underbrace{\frac{1}{3} + \frac{1}{4}}_{\geq 2 \cdot \frac{1}{4} = \frac{1}{2}} + \underbrace{\frac{1}{5} + \frac{1}{6} + \frac{1}{7} + \frac{1}{8}}_{\geq 4 \cdot \frac{1}{8} = \frac{1}{2}} + \cdots$$

Each group sums to at least $1/2$, so $S_{2^k} \geq 1 + k/2 \to \infty$. $\blacksquare$

```
  Why the harmonic series diverges — grouping:

  1   +   1/2   +   (1/3 + 1/4)   +   (1/5 + 1/6 + 1/7 + 1/8)   + ...
  │       │          │                  │
  ≥ 1     ≥ 1/2      ≥ 1/2              ≥ 1/2
  
  Each "block" contributes at least 1/2 → sum grows without bound.
```

---

## 1.4 Telescoping Series

$$\sum_{n=1}^{\infty} (b_n - b_{n+1}) = b_1 - \lim_{n \to \infty} b_n$$

(provided the limit exists).

**Example:** $\sum_{n=1}^{\infty} \frac{1}{n(n+1)} = \sum_{n=1}^{\infty}\left(\frac{1}{n} - \frac{1}{n+1}\right) = 1 - 0 = 1$.

---

## 1.5 The Divergence Test (nth Term Test)

**Theorem 1.4 (Divergence Test).** If $\sum a_n$ converges, then $a_n \to 0$.

Equivalently (contrapositive): **If $a_n \not\to 0$, then $\sum a_n$ diverges.**

**Proof.** $a_n = S_n - S_{n-1} \to S - S = 0$. $\blacksquare$

**⚠ Warning:** $a_n \to 0$ does NOT imply convergence! Example: $\sum 1/n$ — terms $\to 0$ but series diverges.

```
  ┌────────────────────────────────────────────────┐
  │   DIVERGENCE TEST — What it says and doesn't:  │
  ├────────────────────────────────────────────────┤
  │                                                │
  │   aₙ ↛ 0  ══►  Σaₙ diverges    ✅ VALID       │
  │                                                │
  │   aₙ → 0  ══►  Σaₙ converges   ❌ INVALID!    │
  │                 (need other tests)             │
  │                                                │
  │   Σaₙ converges ══► aₙ → 0     ✅ VALID       │
  └────────────────────────────────────────────────┘
```

---

## 1.6 Cauchy Criterion for Series

**Theorem 1.5.** $\sum a_n$ converges if and only if:

$$\forall\, \varepsilon > 0,\; \exists\, N:\; \forall\, m > n \geq N:\; \left|\sum_{k=n+1}^{m} a_k\right| < \varepsilon$$

This is just the Cauchy criterion applied to the partial sums $(S_N)$.

---

## 1.7 The $p$-Series

**Theorem 1.6.** The $p$-series $\sum_{n=1}^{\infty} \frac{1}{n^p}$ converges if and only if $p > 1$.

| $p$ | Series | Converges? | Sum (if known) |
|-----|--------|:----------:|-------|
| $2$ | $\sum 1/n^2$ | ✅ | $\pi^2/6$ |
| $3$ | $\sum 1/n^3$ | ✅ | $\approx 1.202$ (Apéry's constant) |
| $1$ | $\sum 1/n$ | ❌ | Harmonic series |
| $1/2$ | $\sum 1/\sqrt{n}$ | ❌ | Diverges |

---

## 1.8 Linearity of Convergent Series

**Theorem 1.7.** If $\sum a_n = A$ and $\sum b_n = B$, then:
- $\sum (a_n + b_n) = A + B$
- $\sum c \cdot a_n = c \cdot A$ for any constant $c$

---

## 1.9 Real-World Application

- **Signal processing**: Fourier series represent signals as infinite sums of sines/cosines
- **Physics**: Perturbation theory uses series expansions of solutions
- **Finance**: Present value of an annuity is a geometric series
- **Computer science**: Algorithmic complexity analysis often involves bounding series

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Series | Defined as limit of partial sums |
| Geometric series | $\sum r^n = 1/(1-r)$ for $\|r\| < 1$ |
| Harmonic series | $\sum 1/n$ diverges |
| $p$-series | Converges ⟺ $p > 1$ |
| Divergence test | $a_n \not\to 0$ ⟹ $\sum a_n$ diverges |
| Converse fails | $a_n \to 0$ does not imply convergence |
| Cauchy criterion | Tail sums can be made arbitrarily small |
| Telescoping | $\sum(b_n - b_{n+1}) = b_1 - \lim b_n$ |

---

## Quick Revision Questions

1. **Define** convergence of a series in terms of partial sums.

2. **Prove** the geometric series formula $\sum_{n=0}^{\infty} r^n = 1/(1-r)$ for $|r| < 1$.

3. **Prove** the harmonic series diverges using the grouping argument.

4. **True or False:** If $a_n \to 0$, then $\sum a_n$ converges. Justify.

5. **Evaluate** $\sum_{n=1}^{\infty} \frac{1}{n(n+2)}$ using partial fractions and telescoping.

6. **State the Cauchy criterion for series** and explain how it relates to the Cauchy criterion for sequences.

---

[← Previous: Completeness](../02-Sequences/07-completeness.md) | [Back to README](../README.md) | [Next: Comparison Tests →](02-comparison-tests.md)
