# Chapter 2: Upper and Lower Integrals (Darboux Integral)

[← Previous: Riemann Sums](01-riemann-sums.md) | [Back to README](../README.md) | [Next: Riemann Integrability →](03-riemann-integrability.md)

---

## Overview

The **Darboux approach** defines the integral via suprema and infima of upper and lower sums. The **upper integral** $\overline{\int}$ and **lower integral** $\underline{\int}$ always exist for bounded functions, and the function is integrable precisely when they agree.

---

## 2.1 Definitions

**Definition 2.1.** For bounded $f: [a,b] \to \mathbb{R}$:

$$\underline{\int_a^b} f = \sup_P L(f, P) \qquad\text{(lower Darboux integral)}$$

$$\overline{\int_a^b} f = \inf_P U(f, P) \qquad\text{(upper Darboux integral)}$$

where the sup/inf is over all partitions $P$ of $[a,b]$.

**These always exist** for bounded $f$, since:
- $L(f, P) \leq (b-a) \sup|f|$ for all $P$ (bounded above)
- $U(f, P) \geq (b-a) \inf f$ for all $P$ (bounded below)
- $L(f, P_1) \leq U(f, P_2)$ for all $P_1, P_2$ (key inequality)

```
  Upper and lower integrals:

  All U(f,P):  ───U₁───U₂──U₃───U₄──► decreasing
                                        toward inf = ∫̄f
                         ↕ gap
  All L(f,P):  ◄──L₁───L₂──L₃───L₄──  increasing
                                        toward sup = ∫_f

  L(f,P) ≤ ∫_f ≤ ∫̄f ≤ U(f,P)  for ALL P

  Integrable ⟺ ∫_f = ∫̄f  (gap closes)
```

---

## 2.2 The Inequality Chain

**Theorem 2.2.** For any partition $P$:

$$L(f, P) \leq \underline{\int_a^b} f \leq \overline{\int_a^b} f \leq U(f, P)$$

The middle inequality follows from the key lemma: every lower sum $\leq$ every upper sum, so $\sup L \leq \inf U$.

---

## 2.3 Darboux Integrability

**Definition 2.3.** $f$ is **Darboux integrable** (= Riemann integrable) on $[a,b]$ if:

$$\underline{\int_a^b} f = \overline{\int_a^b} f$$

The common value is denoted $\int_a^b f(x)\,dx$ or $\int_a^b f$.

---

## 2.4 ε-Criterion (Darboux)

**Theorem 2.4.** $f$ is Riemann integrable on $[a,b]$ if and only if:

$$\forall\, \varepsilon > 0,\; \exists\, P: \; U(f, P) - L(f, P) < \varepsilon$$

**Proof ($\Rightarrow$).** $\overline{\int} f = \underline{\int} f = I$. For $\varepsilon > 0$, $\exists\, P_1$ with $U(f, P_1) < I + \varepsilon/2$ and $\exists\, P_2$ with $L(f, P_2) > I - \varepsilon/2$. Let $P = P_1 \cup P_2$. Then $U(f, P) \leq U(f, P_1) < I + \varepsilon/2$ and $L(f, P) \geq L(f, P_2) > I - \varepsilon/2$. So $U - L < \varepsilon$. $\blacksquare$

**Proof ($\Leftarrow$).** $0 \leq \overline{\int}f - \underline{\int}f \leq U(f, P) - L(f, P) < \varepsilon$ for all $\varepsilon > 0$. So $\overline{\int} = \underline{\int}$. $\blacksquare$

---

## 2.5 Equivalence: Darboux = Riemann

**Theorem 2.5.** The Darboux and Riemann definitions of integrability are equivalent: $f$ is Darboux integrable $\iff$ $f$ is Riemann integrable (i.e., Riemann sums converge to the same value as $\|P\| \to 0$).

---

## 2.6 Worked Example

**$f(x) = x$ on $[0, 1]$ with uniform partition $P_n$: $x_k = k/n$.**

$$L(f, P_n) = \sum_{k=1}^n \frac{k-1}{n} \cdot \frac{1}{n} = \frac{1}{n^2}\sum_{k=0}^{n-1} k = \frac{(n-1)n}{2n^2} = \frac{n-1}{2n}$$

$$U(f, P_n) = \sum_{k=1}^n \frac{k}{n} \cdot \frac{1}{n} = \frac{n+1}{2n}$$

$$U - L = \frac{1}{n} \to 0 \quad \Rightarrow \quad \int_0^1 x\,dx = \lim \frac{n-1}{2n} = \frac{1}{2}$$

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Lower integral $\underline{\int}$ | $\sup_P L(f, P)$ |
| Upper integral $\overline{\int}$ | $\inf_P U(f, P)$ |
| Always exist | For bounded $f$; $\underline{\int} \leq \overline{\int}$ |
| Integrable | $\underline{\int} = \overline{\int}$ |
| ε-criterion | $\exists\, P$: $U(f,P) - L(f,P) < \varepsilon$ |
| Darboux = Riemann | Both definitions equivalent |

---

## Quick Revision Questions

1. **Define** the upper and lower Darboux integrals. Why do they always exist?

2. **Prove** the ε-criterion: $f$ integrable $\iff$ $\forall \varepsilon$, $\exists P$: $U - L < \varepsilon$.

3. **Compute** $\int_0^1 x^2\,dx$ using upper and lower sums with uniform partitions.

4. **Show** the Dirichlet function is NOT Riemann integrable using upper and lower sums.

5. **Prove** $\underline{\int} f \leq \overline{\int} f$ for any bounded $f$.

---

[← Previous: Riemann Sums](01-riemann-sums.md) | [Back to README](../README.md) | [Next: Riemann Integrability →](03-riemann-integrability.md)
