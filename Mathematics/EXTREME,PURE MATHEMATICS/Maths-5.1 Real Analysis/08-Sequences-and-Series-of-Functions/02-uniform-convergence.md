# Chapter 2: Uniform Convergence

[← Previous: Pointwise Convergence](01-pointwise-convergence.md) | [Back to README](../README.md) | [Next: Weierstrass M-Test →](03-weierstrass-m-test.md)

---

## Overview

**Uniform convergence** strengthens pointwise convergence by requiring a **single** $N$ that works for **all** $x$ simultaneously. This is the key condition that allows us to interchange limits with continuity, integration, and (with care) differentiation.

---

## 2.1 Definition

**Definition 2.1.** $(f_n)$ **converges uniformly** to $f$ on $A$ if:

$$\forall\, \varepsilon > 0,\; \exists\, N \in \mathbb{N}: \; n \geq N \Rightarrow |f_n(x) - f(x)| < \varepsilon \;\;\forall\, x \in A$$

**Equivalently:** $\|f_n - f\|_\infty = \sup_{x \in A}|f_n(x) - f(x)| \to 0$.

```
  Quantifier comparison:

  Pointwise:   ∀ε > 0,  ∀x ∈ A,  ∃N(x,ε):  n ≥ N ⟹ |fₙ(x) - f(x)| < ε
                                  ↑ depends on x

  Uniform:     ∀ε > 0,  ∃N(ε):  ∀x ∈ A,   n ≥ N ⟹ |fₙ(x) - f(x)| < ε
                         ↑ ONE N for ALL x

  Uniform ⟹ Pointwise  (but NOT the reverse)
```

**Geometric picture:**

```
  Uniform convergence means fₙ eventually lies
  within an ε-band around f:

  f(x)+ε ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
           ╱╲   ╱╲    ← fₙ trapped
  f(x)   ╱──╲─╱──╲───────── f(x)
         ╱    ╲    ╲   ← fₙ trapped
  f(x)-ε ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─

  The entire graph of fₙ fits in the band for n ≥ N.
```

---

## 2.2 Sup-Norm Criterion

**Theorem 2.2.** $f_n \to f$ uniformly on $A$ $\iff$ $\|f_n - f\|_\infty \to 0$, i.e.:

$$\sup_{x \in A}|f_n(x) - f(x)| \to 0 \text{ as } n \to \infty$$

**To show NOT uniform:** find a sequence $x_n$ such that $|f_n(x_n) - f(x_n)| \not\to 0$.

---

## 2.3 Examples

**Example 1.** $f_n(x) = x^n$ on $[0, 1)$: pointwise limit $f = 0$.

$$\sup_{x \in [0,1)} x^n = \lim_{x \to 1^-} x^n = 1 \neq 0$$

NOT uniform on $[0, 1)$. (But uniform on $[0, a]$ for any $a < 1$.)

**Example 2.** $f_n(x) = x/n$ on $[0, 1]$: $\sup|x/n| = 1/n \to 0$. Uniform. ✓

**Example 3.** $f_n(x) = \frac{nx}{1+n^2x^2}$: $f_n(1/n) = 1/2$, so $\sup|f_n| \geq 1/2$. NOT uniform. ✗

**Example 4.** $f_n(x) = x^n/n$ on $[0, 1]$: $\sup|x^n/n| = 1/n \to 0$. Uniform. ✓

---

## 2.4 The Interchange Theorems

### Theorem 2.4.1 (Uniform Limit of Continuous = Continuous)

If $f_n \in C(A)$ and $f_n \to f$ uniformly on $A$, then $f \in C(A)$.

**Proof.** Fix $c \in A$, $\varepsilon > 0$. Choose $N$: $\|f_N - f\|_\infty < \varepsilon/3$. Since $f_N$ continuous at $c$, $\exists\, \delta$: $|x - c| < \delta \Rightarrow |f_N(x) - f_N(c)| < \varepsilon/3$.

$$|f(x) - f(c)| \leq |f(x) - f_N(x)| + |f_N(x) - f_N(c)| + |f_N(c) - f(c)|$$
$$< \frac{\varepsilon}{3} + \frac{\varepsilon}{3} + \frac{\varepsilon}{3} = \varepsilon \quad \blacksquare$$

```
  The 3ε trick:

  f(x) ←─ε/3─→ fₙ(x) ←─ε/3─→ fₙ(c) ←─ε/3─→ f(c)
   │              │                │            │
   uniform        continuous       uniform
   convergence    at c             convergence
```

### Theorem 2.4.2 (Interchange of Limit and Integral)

If $f_n \in \mathcal{R}[a,b]$ and $f_n \to f$ uniformly, then $f \in \mathcal{R}[a,b]$ and:

$$\lim_{n\to\infty}\int_a^b f_n = \int_a^b \lim_{n\to\infty} f_n = \int_a^b f$$

**Proof.** $\left|\int_a^b f_n - \int_a^b f\right| \leq \int_a^b|f_n - f| \leq (b-a)\|f_n - f\|_\infty \to 0$. $\blacksquare$

### Theorem 2.4.3 (Interchange of Limit and Derivative)

If $f_n$ differentiable on $[a,b]$, $f_n(x_0) \to A$ for some $x_0$, and $f_n' \to g$ **uniformly**, then $f_n \to f$ uniformly for some $f$, and:

$$f'(x) = g(x) = \lim_{n\to\infty} f_n'(x)$$

**Note:** The key hypothesis is that the **derivatives** converge uniformly (not the functions themselves).

---

## 2.5 Truth Table: When Can We Interchange?

| Operation | Pointwise | Uniform | Extra conditions |
|-----------|-----------|---------|-----------------|
| $\lim f_n$ continuous? | No | **Yes** | — |
| $\lim \int f_n = \int \lim f_n$? | No | **Yes** | — |
| $\lim f_n' = (\lim f_n)'$? | No | Not sufficient | **$f_n'$ uniform + $f_n(x_0)$ conv.** |

---

## 2.6 Cauchy Criterion for Uniform Convergence

**Theorem 2.6.** $(f_n)$ converges uniformly on $A$ iff:

$$\forall\, \varepsilon > 0,\; \exists\, N: \; m, n \geq N \Rightarrow \|f_m - f_n\|_\infty < \varepsilon$$

Useful when we don't know the limit in advance.

---

## 2.7 Dini's Theorem

**Theorem 2.7 (Dini).** If $f_n \in C[a,b]$, $f_n \to f$ pointwise with $f$ continuous, and $(f_n)$ is **monotone** ($f_1 \geq f_2 \geq \cdots$ or $f_1 \leq f_2 \leq \cdots$), then $f_n \to f$ **uniformly**.

**Conditions:** compact domain + continuous limit + monotone sequence.

---

## Summary Table

| Concept | Details |
|---------|---------|
| Uniform convergence | $\sup\|f_n - f\| \to 0$; one $N$ for all $x$ |
| ⟹ Continuity preserved | $f_n$ cont. + uniform $\Rightarrow$ $f$ cont. |
| ⟹ Integral interchange | $\lim\int f_n = \int\lim f_n$ |
| Derivative interchange | Need $f_n'$ uniform + one-point convergence |
| Cauchy criterion | $\|f_m - f_n\|_\infty \to 0$ |
| Dini's theorem | Monotone + compact + cont. limit ⟹ uniform |

---

## Quick Revision Questions

1. **Define** uniform convergence. How does it differ from pointwise convergence?

2. **Prove** the uniform limit of continuous functions is continuous (3ε argument).

3. **Show** $f_n(x) = x^n$ on $[0,1]$ does NOT converge uniformly.

4. **Prove** $f_n \to f$ uniformly $\Rightarrow$ $\int f_n \to \int f$.

5. **State** the precise hypotheses for interchanging $\lim$ and $d/dx$.

6. **State** Dini's theorem and explain why each hypothesis is needed.

---

[← Previous: Pointwise Convergence](01-pointwise-convergence.md) | [Back to README](../README.md) | [Next: Weierstrass M-Test →](03-weierstrass-m-test.md)
