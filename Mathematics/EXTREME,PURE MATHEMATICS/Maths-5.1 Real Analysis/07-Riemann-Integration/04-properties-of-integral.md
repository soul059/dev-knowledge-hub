# Chapter 4: Properties of the Riemann Integral

[← Previous: Riemann Integrability](03-riemann-integrability.md) | [Back to README](../README.md) | [Next: Fundamental Theorem of Calculus →](05-fundamental-theorem-of-calculus.md)

---

## Overview

Once we know a function is integrable, we can derive the algebraic and order properties of the integral: **linearity**, **monotonicity**, **additivity over intervals**, the **triangle inequality**, and more.

---

## 4.1 Linearity

**Theorem 4.1.** If $f, g \in \mathcal{R}[a,b]$ and $c \in \mathbb{R}$, then:

(a) $cf \in \mathcal{R}[a,b]$ and $\int_a^b cf = c\int_a^b f$

(b) $f + g \in \mathcal{R}[a,b]$ and $\int_a^b (f + g) = \int_a^b f + \int_a^b g$

**Proof of (b).** On each $[x_{k-1}, x_k]$:

$$\sup(f+g) \leq \sup f + \sup g, \qquad \inf(f+g) \geq \inf f + \inf g$$

So $\omega_k(f+g) \leq \omega_k(f) + \omega_k(g)$. Given $\varepsilon > 0$, choose $P_1, P_2$ with $U(f,P_1) - L(f,P_1) < \varepsilon/2$ and $U(g,P_2) - L(g,P_2) < \varepsilon/2$. Then $P = P_1 \cup P_2$ satisfies $U(f+g, P) - L(f+g, P) < \varepsilon$. So $f+g$ is integrable.

For the value: $L(f,P) + L(g,P) \leq L(f+g, P) \leq U(f+g,P) \leq U(f,P) + U(g,P)$. Taking limits gives $\int f + \int g = \int(f+g)$. $\blacksquare$

---

## 4.2 Monotonicity (Order)

**Theorem 4.2.** If $f, g \in \mathcal{R}[a,b]$ and $f(x) \leq g(x)$ for all $x \in [a,b]$, then:

$$\int_a^b f \leq \int_a^b g$$

**Proof.** $L(f, P) \leq L(g, P)$ for all $P$ (since $\inf f \leq \inf g$ on each subinterval). Taking sup: $\int f = \sup_P L(f,P) \leq \sup_P L(g,P) = \int g$. $\blacksquare$

**Corollary.** If $m \leq f(x) \leq M$ on $[a,b]$, then $m(b-a) \leq \int_a^b f \leq M(b-a)$.

---

## 4.3 Additivity over Intervals

**Theorem 4.3.** If $a < c < b$, then $f \in \mathcal{R}[a,b]$ iff $f \in \mathcal{R}[a,c]$ and $f \in \mathcal{R}[c,b]$, and:

$$\int_a^b f = \int_a^c f + \int_c^b f$$

**Proof sketch.** ($\Rightarrow$) If $P$ is a partition of $[a,b]$, add $c$ to get $P' = P \cup \{c\}$, which splits as $P'_1$ on $[a,c]$ and $P'_2$ on $[c,b]$. The oscillation sum splits: $\sum \omega_k \Delta x_k = \sum_{P'_1} + \sum_{P'_2}$. Each part $< \varepsilon$ if the total is. $\blacksquare$

```
  Additivity over intervals:

  a─────────────c─────────────b
  ├─────────────┤─────────────┤
  │  ∫ₐᶜ f(x)dx │  ∫ᶜᵇ f(x)dx│
  └─────────────┴─────────────┘
         ∫ₐᵇ f(x)dx = sum
```

**Convention.** We define $\int_a^a f = 0$ and $\int_b^a f = -\int_a^b f$.

---

## 4.4 Triangle Inequality for Integrals

**Theorem 4.4.** If $f \in \mathcal{R}[a,b]$, then $|f| \in \mathcal{R}[a,b]$ and:

$$\left|\int_a^b f\right| \leq \int_a^b |f|$$

**Proof.** $|f|$ integrable: $\omega_k(|f|) \leq \omega_k(f)$ (since $||f(x)| - |f(y)|| \leq |f(x) - f(y)|$), so integrability passes from $f$ to $|f|$.

Inequality: $-|f| \leq f \leq |f|$ implies $-\int|f| \leq \int f \leq \int|f|$ by monotonicity. $\blacksquare$

---

## 4.5 Products and Max/Min

**Theorem 4.5.** If $f, g \in \mathcal{R}[a,b]$, then:

(a) $fg \in \mathcal{R}[a,b]$

(b) $\max(f,g)$ and $\min(f,g)$ are in $\mathcal{R}[a,b]$

**Proof of (a).** First show $f^2 \in \mathcal{R}$: $\omega_k(f^2) \leq 2M\,\omega_k(f)$ where $M = \sup|f|$. Then $fg = \frac{1}{2}[(f+g)^2 - f^2 - g^2]$.

**Proof of (b).** $\max(f,g) = \frac{f+g+|f-g|}{2}$, and $|f-g|$ is integrable by Theorem 4.4 applied to $f-g$. $\blacksquare$

---

## 4.6 Mean Value Theorem for Integrals

**Theorem 4.6 (MVT for Integrals).** If $f \in C[a,b]$, then $\exists\, c \in [a,b]$:

$$\int_a^b f = f(c)(b-a)$$

**Proof.** By EVT, $f$ attains min $m$ and max $M$. By monotonicity: $m(b-a) \leq \int f \leq M(b-a)$. So $m \leq \frac{1}{b-a}\int f \leq M$. By IVT, $\exists\, c$ with $f(c) = \frac{1}{b-a}\int f$. $\blacksquare$

```
  Mean Value Theorem for Integrals:

  f(x)
   │        ╱\
   │      ╱   ╲     ← f(c) = average value
   │    ╱──────╲────── f(c)
   │  ╱  equal  ╲
   │╱   areas    ╲
   └──────────────────── x
   a    c         b

   Shaded area = f(c)·(b-a) = ∫ₐᵇ f
```

---

## 4.7 Weighted Mean Value Theorem

**Theorem 4.7.** If $f \in C[a,b]$ and $g \in \mathcal{R}[a,b]$ with $g(x) \geq 0$, then $\exists\, c \in [a,b]$:

$$\int_a^b fg = f(c) \int_a^b g$$

---

## Summary Table

| Property | Statement |
|----------|-----------|
| Linearity | $\int(cf + g) = c\int f + \int g$ |
| Monotonicity | $f \leq g \Rightarrow \int f \leq \int g$ |
| Additivity | $\int_a^b = \int_a^c + \int_c^b$ |
| Triangle inequality | $|\int f| \leq \int|f|$ |
| Products | $f,g \in \mathcal{R} \Rightarrow fg \in \mathcal{R}$ |
| Integral MVT | $\int_a^b f = f(c)(b-a)$ for some $c$ |
| Bounds | $m(b-a) \leq \int_a^b f \leq M(b-a)$ |

---

## Quick Revision Questions

1. **Prove** linearity: $\int_a^b (f+g) = \int_a^b f + \int_a^b g$.

2. **Prove** the triangle inequality $|\int_a^b f| \leq \int_a^b |f|$.

3. **Prove** the integral MVT for $f \in C[a,b]$.

4. **Show** $fg \in \mathcal{R}[a,b]$ when $f, g \in \mathcal{R}[a,b]$.

5. **Use** monotonicity to show $\int_0^1 \frac{1}{1+x^2}\,dx \leq 1$.

6. **Discuss** whether the converse of "$f \leq g \Rightarrow \int f \leq \int g$" holds.

---

[← Previous: Riemann Integrability](03-riemann-integrability.md) | [Back to README](../README.md) | [Next: Fundamental Theorem of Calculus →](05-fundamental-theorem-of-calculus.md)
