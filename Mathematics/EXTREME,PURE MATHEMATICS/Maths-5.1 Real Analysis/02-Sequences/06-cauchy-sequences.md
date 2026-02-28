# Chapter 6: Cauchy Sequences

[← Previous: Bolzano-Weierstrass Theorem](05-bolzano-weierstrass-theorem.md) | [Back to README](../README.md) | [Next: Completeness →](07-completeness.md)

---

## Overview

Cauchy sequences are a way to describe "convergent behavior" **without knowing the limit in advance**. A sequence is Cauchy if its terms get arbitrarily close to *each other*. In ℝ, Cauchy sequences are precisely the convergent sequences — this is another expression of completeness.

---

## 6.1 Definition

**Definition 6.1 (Cauchy Sequence).** A sequence $(a_n)$ is **Cauchy** if:

$$\forall\, \varepsilon > 0,\; \exists\, N \in \mathbb{N} \;\text{such that}\; \forall\, m, n \geq N:\; |a_m - a_n| < \varepsilon$$

```
  Cauchy vs Convergent — Definition Comparison:

  CONVERGENT: terms get close to a FIXED LIMIT L
  ┌──────────────────────────────────────────────┐
  │ ∀ε > 0, ∃N: ∀n ≥ N,  |aₙ - L| < ε          │
  │                         ▲                     │
  │                      compare                  │
  │                      to L                     │
  └──────────────────────────────────────────────┘

  CAUCHY: terms get close to EACH OTHER
  ┌──────────────────────────────────────────────┐
  │ ∀ε > 0, ∃N: ∀m,n ≥ N,  |aₘ - aₙ| < ε       │
  │                          ▲                    │
  │                       compare                 │
  │                       to each                 │
  │                       other                   │
  └──────────────────────────────────────────────┘

  Key advantage: No L needed! We can verify "Cauchy-ness"
  without knowing the limit.
```

---

## 6.2 Convergent ⟹ Cauchy

**Theorem 6.2.** Every convergent sequence is Cauchy.

**Proof.** Suppose $a_n \to L$. Let $\varepsilon > 0$. Choose $N$ with $|a_n - L| < \varepsilon/2$ for $n \geq N$.

For $m, n \geq N$:
$$|a_m - a_n| = |a_m - L + L - a_n| \leq |a_m - L| + |a_n - L| < \frac{\varepsilon}{2} + \frac{\varepsilon}{2} = \varepsilon$$
$\blacksquare$

---

## 6.3 Cauchy ⟹ Bounded

**Theorem 6.3.** Every Cauchy sequence is bounded.

**Proof.** Take $\varepsilon = 1$. There exists $N$ with $|a_m - a_n| < 1$ for $m, n \geq N$. In particular, $|a_n - a_N| < 1$ for $n \geq N$, so $|a_n| < |a_N| + 1$ for $n \geq N$.

Let $M = \max(|a_1|, \ldots, |a_{N-1}|, |a_N| + 1)$. Then $|a_n| \leq M$ for all $n$. $\blacksquare$

---

## 6.4 Cauchy ⟹ Convergent (in ℝ)

**Theorem 6.4 (Cauchy Completeness of ℝ).** Every Cauchy sequence in $\mathbb{R}$ converges.

**Proof.**

**Step 1:** $(a_n)$ is Cauchy, hence bounded (Theorem 6.3).

**Step 2:** By Bolzano-Weierstrass, $(a_n)$ has a convergent subsequence $(a_{n_k}) \to L$.

**Step 3:** We show $a_n \to L$. Let $\varepsilon > 0$.
- Since $(a_n)$ is Cauchy, $\exists\, N_1$ with $|a_m - a_n| < \varepsilon/2$ for $m, n \geq N_1$.
- Since $a_{n_k} \to L$, $\exists\, K$ with $|a_{n_K} - L| < \varepsilon/2$ and $n_K \geq N_1$.

For $n \geq N_1$:
$$|a_n - L| \leq |a_n - a_{n_K}| + |a_{n_K} - L| < \frac{\varepsilon}{2} + \frac{\varepsilon}{2} = \varepsilon$$
$\blacksquare$

```
  Proof Structure:

  Cauchy ───► Bounded ───► BW gives convergent
    │                      subsequence aₙₖ → L
    │                              │
    │                              ▼
    └──────────────────────► Combine: Cauchy + subseq → L
                              gives full sequence → L

  Key trick: A Cauchy sequence with a convergent
  subsequence must itself converge to the same limit.
```

---

## 6.5 The Complete Picture

**Theorem 6.5.** In $\mathbb{R}$: $(a_n)$ converges $\iff$ $(a_n)$ is Cauchy.

```
  In ℝ:     Convergent ═══════► Cauchy
                 ▲                  │
                 │                  │
                 │    (requires     │
                 │   completeness)  │
                 │                  │
                 ◄══════════════════╝

  In ℚ:     Convergent ═══════► Cauchy
                                    │
                                    │  (NOT reversible!)
                                    ▼
                              aₙ = rational approx. of √2
                              is Cauchy in ℚ but does NOT
                              converge in ℚ
```

---

## 6.6 Why Cauchy Sequences Matter

| Situation | Why Cauchy Criterion Helps |
|-----------|--------------------------|
| **Unknown limit** | Verify convergence without knowing $L$ |
| **Completeness** | Defines what "complete" means for metric spaces |
| **Series convergence** | $\sum a_n$ converges ⟺ partial sums are Cauchy |
| **Function spaces** | Banach spaces = complete normed vector spaces |
| **Construction of ℝ** | ℝ can be constructed as completion of ℚ via Cauchy sequences |

---

## 6.7 Cauchy Criterion for Series

**Theorem 6.6.** $\sum_{n=1}^{\infty} a_n$ converges if and only if:

$$\forall\, \varepsilon > 0,\; \exists\, N:\; \forall\, m > n \geq N:\; |a_{n+1} + a_{n+2} + \cdots + a_m| < \varepsilon$$

**Corollary 6.7 (Divergence Test).** If $\sum a_n$ converges, then $a_n \to 0$.

**Proof.** Set $m = n + 1$ in the Cauchy criterion: $|a_{n+1}| < \varepsilon$ for $n \geq N$. $\blacksquare$

---

## 6.8 Contractive Sequences

**Definition 6.8.** A sequence $(a_n)$ is **contractive** if there exists $0 < c < 1$ with:
$$|a_{n+2} - a_{n+1}| \leq c \cdot |a_{n+1} - a_n| \quad \text{for all } n$$

**Theorem 6.9.** Every contractive sequence is Cauchy (hence convergent in ℝ).

**Proof.** Let $d = |a_2 - a_1|$. By induction: $|a_{n+1} - a_n| \leq c^{n-1} d$. For $m > n$:
$$|a_m - a_n| \leq \sum_{k=n}^{m-1} |a_{k+1} - a_k| \leq d \sum_{k=n}^{m-1} c^{k-1} \leq \frac{d \cdot c^{n-1}}{1-c}$$

Since $c^{n-1} \to 0$, this can be made $< \varepsilon$. $\blacksquare$

```
  Contractive sequence — gaps shrink geometrically:

  │a₂-a₁│ = d
  │a₃-a₂│ ≤ cd
  │a₄-a₃│ ≤ c²d       c < 1, so gaps shrink
  │a₅-a₄│ ≤ c³d       ────────────►
  ...
  │aₙ₊₁-aₙ│ ≤ cⁿ⁻¹d → 0
```

---

## 6.9 Real-World Application

- **Fixed-point iteration**: Newton's method produces (under good conditions) a contractive sequence converging to a root
- **Signal processing**: Successive approximation ADCs produce Cauchy sequences of voltage approximations
- **Constructing ℝ from ℚ**: Equivalence classes of Cauchy sequences in ℚ form the real numbers

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Cauchy sequence | $\|a_m - a_n\| < \varepsilon$ for large $m, n$ |
| Convergent ⟹ Cauchy | Triangle inequality proof |
| Cauchy ⟹ bounded | Similar to convergent ⟹ bounded |
| Cauchy ⟹ convergent (in ℝ) | Uses BW + subsequence trick |
| Cauchy completeness | In ℝ: Cauchy ⟺ convergent |
| Fails in ℚ | Rational Cauchy sequences can "converge to irrationals" |
| Cauchy criterion for series | Partial sums must be Cauchy |
| Contractive sequences | Geometric shrinking of gaps ⟹ Cauchy |

---

## Quick Revision Questions

1. **State the definition** of a Cauchy sequence. Compare it to the definition of convergence.

2. **Prove:** Every convergent sequence is Cauchy. Is the converse true in ℚ?

3. **Prove:** Every Cauchy sequence in ℝ is convergent. Which results from previous chapters are used?

4. **Give an example** of a Cauchy sequence in $\mathbb{Q}$ that does not converge in $\mathbb{Q}$.

5. **Prove the Divergence Test:** If $\sum a_n$ converges, then $a_n \to 0$. Use the Cauchy criterion.

6. **Show that the sequence $a_1 = 1$, $a_{n+1} = \frac{1}{2}a_n + 1$** is contractive and find its limit.

---

[← Previous: Bolzano-Weierstrass Theorem](05-bolzano-weierstrass-theorem.md) | [Back to README](../README.md) | [Next: Completeness →](07-completeness.md)
