# Chapter 2: Comparison Tests

[← Previous: Convergence of Series](01-convergence-of-series.md) | [Back to README](../README.md) | [Next: Ratio and Root Tests →](03-ratio-and-root-tests.md)

---

## Overview

Comparison tests allow us to determine convergence or divergence of a series by comparing it to a series whose behavior we already know. These are the most fundamental convergence tests.

---

## 2.1 Direct Comparison Test

**Theorem 2.1 (Direct Comparison Test).** Suppose $0 \leq a_n \leq b_n$ for all $n \geq N_0$.

- If $\sum b_n$ converges, then $\sum a_n$ converges.
- If $\sum a_n$ diverges, then $\sum b_n$ diverges.

**Proof.** The partial sums $S_N = \sum_{n=1}^N a_n$ are increasing (since $a_n \geq 0$) and bounded above (since $S_N \leq \sum_{n=1}^N b_n \leq \sum_{n=1}^{\infty} b_n$). By MCT, $(S_N)$ converges. $\blacksquare$

```
  Direct Comparison — Logic:

  0 ≤ aₙ ≤ bₙ

  Σbₙ converges (bigger series)
       │
       ▼
  Σaₙ converges (smaller series is "squeezed")

  Σaₙ diverges (smaller series)
       │
       ▼
  Σbₙ diverges (bigger series must also diverge)
```

### Common Comparison Partners

| Known series | Convergence | Use as... |
|-------------|:-----------:|-----------|
| $\sum 1/n^p$, $p > 1$ | ✅ | Upper bound (to prove convergence) |
| $\sum 1/n$ | ❌ | Lower bound (to prove divergence) |
| $\sum r^n$, $\|r\| < 1$ | ✅ | Upper bound |
| $\sum 1/n^p$, $p \leq 1$ | ❌ | Lower bound |

### Example

$\sum \frac{1}{n^2 + 3n}$ converges because $\frac{1}{n^2 + 3n} < \frac{1}{n^2}$ and $\sum 1/n^2$ converges.

---

## 2.2 Limit Comparison Test

**Theorem 2.2 (Limit Comparison Test).** Let $a_n, b_n > 0$ for all $n$ and suppose:

$$L = \lim_{n \to \infty} \frac{a_n}{b_n}$$

- If $0 < L < \infty$: $\sum a_n$ converges $\iff$ $\sum b_n$ converges
- If $L = 0$ and $\sum b_n$ converges: $\sum a_n$ converges
- If $L = \infty$ and $\sum b_n$ diverges: $\sum a_n$ diverges

**Proof (case $0 < L < \infty$).** Choose $\varepsilon = L/2$. Then for large $n$:
$$\frac{L}{2} < \frac{a_n}{b_n} < \frac{3L}{2}$$
So $\frac{L}{2} b_n < a_n < \frac{3L}{2} b_n$. Apply the direct comparison test. $\blacksquare$

```
  Limit Comparison — When to use:

  ┌─────────────────────────────────────────────┐
  │ Use LIMIT comparison when:                   │
  │                                               │
  │ • Direct comparison is hard (terms aren't    │
  │   cleanly ≤ or ≥)                            │
  │ • aₙ and bₙ have "similar growth rate"       │
  │ • aₙ/bₙ → finite nonzero constant           │
  │                                               │
  │ Example: aₙ = (2n+1)/(3n³-n+7)              │
  │ Compare with bₙ = 1/n² (same growth rate)   │
  │ aₙ/bₙ → 2/3 ∈ (0,∞), so both converge.     │
  └─────────────────────────────────────────────┘
```

### Example

$\sum \frac{\sqrt{n}}{n^2 + 1}$: Compare with $b_n = 1/n^{3/2}$.

$$\frac{a_n}{b_n} = \frac{\sqrt{n} \cdot n^{3/2}}{n^2 + 1} = \frac{n^2}{n^2 + 1} \to 1$$

Since $\sum 1/n^{3/2}$ converges ($p = 3/2 > 1$), so does $\sum \frac{\sqrt{n}}{n^2 + 1}$.

---

## 2.3 Cauchy Condensation Test

**Theorem 2.3 (Cauchy Condensation Test).** If $(a_n)$ is decreasing and $a_n \geq 0$, then:

$$\sum_{n=1}^{\infty} a_n \;\text{converges} \iff \sum_{k=0}^{\infty} 2^k a_{2^k} \;\text{converges}$$

**Proof idea.** Group terms in blocks of size $2^k$:
$$\sum_{n=2^k}^{2^{k+1}-1} a_n \leq 2^k a_{2^k} \leq 2 \sum_{n=2^{k-1}}^{2^k - 1} a_n$$

The condensed series has comparable partial sums. $\blacksquare$

### Application: $p$-Series Test

For $a_n = 1/n^p$: $2^k a_{2^k} = 2^k / (2^k)^p = 2^{k(1-p)} = (2^{1-p})^k$.

This geometric series converges iff $2^{1-p} < 1$, i.e., $p > 1$.

---

## 2.4 Decision Flowchart

```
  Which comparison test to use?
  ═════════════════════════════

  Can you find bₙ with aₙ ≤ bₙ (or aₙ ≥ bₙ) directly?
     │
     ├── YES → Use DIRECT Comparison Test
     │
     └── NO → Can you compute lim(aₙ/bₙ)?
              │
              ├── YES, limit is finite & nonzero → LIMIT Comparison
              │
              └── Is (aₙ) decreasing?
                   │
                   └── YES → Consider CONDENSATION Test
```

---

## Summary Table

| Test | Condition | Conclusion |
|------|-----------|------------|
| Direct comparison | $0 \leq a_n \leq b_n$, $\sum b_n$ conv. | $\sum a_n$ converges |
| Direct comparison | $0 \leq b_n \leq a_n$, $\sum b_n$ div. | $\sum a_n$ diverges |
| Limit comparison | $a_n/b_n \to L \in (0,\infty)$ | Same behavior as $\sum b_n$ |
| Condensation | $a_n \downarrow 0$ | $\sum a_n$ conv. ⟺ $\sum 2^k a_{2^k}$ conv. |

---

## Quick Revision Questions

1. **State and prove the Direct Comparison Test.** Why must $a_n \geq 0$?

2. **Use limit comparison** to determine convergence of $\sum \frac{n}{n^3 + 5}$.

3. **Prove the $p$-series test** ($\sum 1/n^p$ converges iff $p > 1$) using the Condensation Test.

4. **Does $\sum \frac{1}{\sqrt{n^3 + n}}$ converge?** Which test did you use?

5. **True or False:** The Limit Comparison Test works even when $a_n$ can be negative.

6. **Determine convergence:** $\sum \frac{\ln n}{n^2}$.

---

[← Previous: Convergence of Series](01-convergence-of-series.md) | [Back to README](../README.md) | [Next: Ratio and Root Tests →](03-ratio-and-root-tests.md)
