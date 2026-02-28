# Chapter 5.3 — Integral Domains

[← Previous: Subrings](02-subrings.md) | [Back to README](../README.md) | [Next: Fields →](04-fields.md)

---

## Overview

An **integral domain** is a commutative ring with no zero divisors — generalising $\mathbb{Z}$. The cancellation law holds, and we can build a field of fractions. Integral domains are the natural setting for factorisation theory.

---

## 1. Definition

**Definition.** An **integral domain** (or simply **domain**) is a commutative ring $R$ with $1 \neq 0$ such that:

$$ab = 0 \implies a = 0 \text{ or } b = 0$$

Equivalently: $R$ has **no zero divisors**.

```
  Integral Domain:
  
  Commutative ring +  1 ≠ 0  +  no zero divisors
  
  ab = 0  ⟹  a = 0 or b = 0
  
  Examples:    ℤ, ℚ, ℝ, ℂ, ℤ[x], ℤ[i], ℤ[√d], F[x]
  Non-examples: ℤ₆, M₂(ℝ), ℤ × ℤ
```

---

## 2. The Cancellation Law

**Theorem 5.5.** In an integral domain $R$: if $a \neq 0$ and $ab = ac$, then $b = c$.

*Proof.* $ab = ac \implies a(b - c) = 0$. Since $a \neq 0$ and $R$ has no zero divisors, $b - c = 0$, so $b = c$. $\blacksquare$

**Converse.** A commutative ring satisfying the cancellation law is an integral domain.

*Proof.* If $ab = 0$ and $a \neq 0$, then $ab = a \cdot 0$, so $b = 0$ by cancellation. $\blacksquare$

---

## 3. Examples and Non-Examples

| Ring | Integral domain? | Reason |
|------|------------------|--------|
| $\mathbb{Z}$ | ✓ | Prototype |
| $\mathbb{Q}, \mathbb{R}, \mathbb{C}$ | ✓ | Every field is a domain |
| $\mathbb{Z}[x]$ | ✓ | No zero divisors in polynomial ring over domain |
| $\mathbb{Z}[i]$ | ✓ | Gaussian integers; norm argument |
| $\mathbb{Z}_p$ ($p$ prime) | ✓ | Field |
| $\mathbb{Z}_6$ | ✗ | $2 \cdot 3 = 0$ |
| $\mathbb{Z}_4$ | ✗ | $2 \cdot 2 = 0$ |
| $M_2(\mathbb{R})$ | ✗ | Non-commutative; has zero divisors |
| $\mathbb{Z} \times \mathbb{Z}$ | ✗ | $(1,0)(0,1) = (0,0)$ |
| $\mathbb{R}[x]/(x^2)$ | ✗ | $\bar{x}^2 = 0$; nilpotent |

---

## 4. $\mathbb{Z}_n$ Is an Integral Domain iff $n$ Is Prime

**Theorem 5.6.** $\mathbb{Z}_n$ is an integral domain if and only if $n$ is prime.

*Proof.*

$(\Rightarrow)$ If $n = ab$ with $1 < a, b < n$, then $\bar{a} \cdot \bar{b} = \overline{ab} = \bar{0}$ but $\bar{a}, \bar{b} \neq \bar{0}$. So $\mathbb{Z}_n$ has zero divisors.

$(\Leftarrow)$ If $n = p$ is prime and $\bar{a}\bar{b} = \bar{0}$, then $p \mid ab$. By Euclid's lemma, $p \mid a$ or $p \mid b$, so $\bar{a} = \bar{0}$ or $\bar{b} = \bar{0}$. $\blacksquare$

---

## 5. Finite Integral Domains Are Fields

**Theorem 5.7.** Every finite integral domain is a field.

*Proof.* Let $R$ be a finite integral domain and $a \neq 0$. Consider the map $\phi_a: R \to R$ defined by $\phi_a(r) = ar$.

By cancellation, $\phi_a$ is injective. Since $R$ is finite, $\phi_a$ is also surjective.

So $1 \in \text{im}(\phi_a)$: there exists $b$ with $ab = 1$. Thus $a$ is a unit. $\blacksquare$

```
  Finite integral domain ⟹ field
  
  Key idea:  injective + finite domain ⟹ surjective
  
  a ≠ 0  ⟹  r ↦ ar is injective (cancellation)
         ⟹  r ↦ ar is surjective (finite set!)
         ⟹  ∃b: ab = 1
         ⟹  a is a unit
```

**Corollary.** $\mathbb{Z}_p$ is a field for every prime $p$.

---

## 6. Characteristic of an Integral Domain

**Definition.** The **characteristic** of a ring $R$, denoted $\text{char}(R)$, is:

$$\text{char}(R) = \min\{n \in \mathbb{Z}^+ : n \cdot 1 = 0\}$$

or $0$ if no such $n$ exists (where $n \cdot 1 = \underbrace{1 + 1 + \cdots + 1}_{n}$).

**Theorem 5.8.** The characteristic of an integral domain is either $0$ or a prime $p$.

*Proof.* Suppose $\text{char}(R) = n = ab$ with $1 < a, b < n$. Then $0 = n \cdot 1 = (a \cdot 1)(b \cdot 1)$. Since $R$ is a domain, $a \cdot 1 = 0$ or $b \cdot 1 = 0$, contradicting minimality of $n$. $\blacksquare$

---

## 7. Field of Fractions

**Theorem 5.9.** Every integral domain $R$ embeds into a field $\text{Frac}(R)$, called its **field of fractions**.

**Construction.** On $R \times (R \setminus \{0\})$, define:

$$(a, b) \sim (c, d) \iff ad = bc$$

Write $\frac{a}{b}$ for the equivalence class. Define:

$$\frac{a}{b} + \frac{c}{d} = \frac{ad + bc}{bd}, \qquad \frac{a}{b} \cdot \frac{c}{d} = \frac{ac}{bd}$$

```
  Field of Fractions:
  
  Frac(ℤ)     = ℚ
  Frac(ℤ[i])  = ℚ(i)    = {a + bi : a,b ∈ ℚ}
  Frac(F[x])  = F(x)    = rational functions
  Frac(ℤ[√2]) = ℚ(√2)   = {a + b√2 : a,b ∈ ℚ}
  
  R ↪ Frac(R) via  r ↦ r/1
  
  Universal property: any injective ring homomorphism
  R → F (F a field) factors through Frac(R).
```

**Theorem 5.10 (Universal Property).** If $\varphi: R \hookrightarrow F$ is an injective ring homomorphism into a field $F$, then $\varphi$ extends uniquely to $\tilde{\varphi}: \text{Frac}(R) \to F$.

$$\tilde{\varphi}\left(\frac{a}{b}\right) = \varphi(a) \cdot \varphi(b)^{-1}$$

---

## 8. Characteristic and Prime Subring

Every ring $R$ has a unique ring homomorphism $\mathbb{Z} \to R$ sending $n \mapsto n \cdot 1_R$.

- If $\text{char}(R) = 0$: the image is $\cong \mathbb{Z}$ (the **prime subring**).
- If $\text{char}(R) = p$: the image is $\cong \mathbb{Z}_p$.

For an integral domain:

| $\text{char}(R)$ | Prime subring | $\text{Frac}(\text{prime subring})$ |
|-------------------|--------------|-------------------------------------|
| $0$ | $\cong \mathbb{Z}$ | $\cong \mathbb{Q}$ |
| $p$ | $\cong \mathbb{F}_p$ | $\cong \mathbb{F}_p$ (already a field) |

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| Integral domain | Commutative ring, $1 \neq 0$, no zero divisors |
| Cancellation | $a \neq 0, ab = ac \implies b = c$ |
| $\mathbb{Z}_n$ is a domain | $\iff n$ is prime |
| Finite domain | $\implies$ field |
| $\text{char}(R)$ | $= 0$ or prime $p$ (for domains) |
| Field of fractions | $R \hookrightarrow \text{Frac}(R)$; e.g., $\mathbb{Z} \hookrightarrow \mathbb{Q}$ |

---

## Quick Revision Questions

1. **Prove** the cancellation law for integral domains.

2. **Show** that $\mathbb{Z}[\sqrt{-5}]$ is an integral domain but not a UFD (hint: $6 = 2 \cdot 3 = (1+\sqrt{-5})(1-\sqrt{-5})$).

3. **Determine** $\text{char}(\mathbb{Z}_7)$ and $\text{char}(\mathbb{Q})$.

4. **Construct** $\text{Frac}(\mathbb{Z}[\sqrt{2}])$ explicitly.

5. **Prove** that a finite integral domain is a field.

6. **Find** all zero divisors in $\mathbb{Z}_{15}$.

---

[← Previous: Subrings](02-subrings.md) | [Back to README](../README.md) | [Next: Fields →](04-fields.md)
