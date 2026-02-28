# Chapter 7.4 — Polynomial Rings

[← Previous: Euclidean Domains](03-euclidean-domains.md) | [Back to README](../README.md) | [Next: Irreducibility Criteria →](05-irreducibility-criteria.md)

---

## Overview

Polynomial rings $R[x]$ are the most important examples of ring extensions. We study their algebraic structure, division properties, content and primitivity, and the fundamental result that **$R$ is a UFD $\implies$ $R[x]$ is a UFD** (Gauss's theorem).

---

## 1. Construction of $R[x]$

**Definition.** For a commutative ring $R$, the **polynomial ring** $R[x]$ consists of formal expressions

$$f(x) = a_n x^n + a_{n-1} x^{n-1} + \cdots + a_1 x + a_0$$

where $a_i \in R$, with addition and multiplication defined by:

$$(f + g)(x) = \sum (a_i + b_i) x^i, \qquad (fg)(x) = \sum_{k} \left(\sum_{i+j=k} a_i b_j\right) x^k$$

**Degree:** $\deg(f) = $ largest $n$ with $a_n \neq 0$. Leading coefficient is $a_n$.

$f$ is **monic** if $a_n = 1$.

$f$ is a **unit** in $R[x]$ $\iff$ $a_0$ is a unit in $R$ and all other $a_i$ are nilpotent. (If $R$ is a domain, units of $R[x]$ are exactly units of $R$.)

---

## 2. Degree Arithmetic

**Theorem 7.7.** If $R$ is an integral domain and $f, g \in R[x]$:

$$\deg(fg) = \deg(f) + \deg(g)$$

*Proof.* Leading coefficient of $fg$ is the product of leading coefficients, which is non-zero since $R$ is a domain. $\blacksquare$

**Corollary.** If $R$ is a domain, then $R[x]$ is a domain.

**Warning:** Over $\mathbb{Z}/6\mathbb{Z}$: $(2x+1)(3x+1) = 6x^2 + 5x + 1 = 5x + 1$. Degree dropped!

---

## 3. Division Algorithm over Fields

**Theorem 7.8 (Division Algorithm).** If $F$ is a field and $f, g \in F[x]$ with $g \neq 0$, there exist unique $q, r \in F[x]$ such that:

$$f = gq + r \quad \text{with } r = 0 \text{ or } \deg(r) < \deg(g)$$

This makes $F[x]$ a Euclidean domain with $\delta = \deg$.

**Important:** The division algorithm works over $F[x]$ (fields) but not over $\mathbb{Z}[x]$ in general.

```
  Division: f(x) = x³ + 2x + 1 by g(x) = x + 1 in ℚ[x]
  
            x² - x + 3
          _______________
  x + 1 | x³ + 0x² + 2x + 1
          x³ + x²
          ────────
              -x² + 2x
              -x² - x
              ────────
                   3x + 1
                   3x + 3
                   ──────
                      -2
  
  x³ + 2x + 1 = (x+1)(x²-x+3) + (-2)
```

---

## 4. Roots and the Factor Theorem

**Theorem 7.9 (Remainder Theorem).** $f(a) = $ remainder when dividing $f(x)$ by $(x-a)$.

**Corollary (Factor Theorem).** $a$ is a root of $f \iff (x-a) \mid f(x)$.

**Theorem 7.10.** If $R$ is an integral domain and $f \in R[x]$ has degree $n$, then $f$ has at most $n$ roots in $R$.

**Warning:** Over $\mathbb{Z}/8\mathbb{Z}$: $x^2 - 1 = 0$ has four roots $\{1, 3, 5, 7\}$ — the theorem fails for non-domains.

---

## 5. Content and Primitive Polynomials

**Definition.** For $f = a_n x^n + \cdots + a_0 \in \mathbb{Z}[x]$:

- The **content** of $f$ is $c(f) = \gcd(a_0, a_1, \ldots, a_n)$.
- $f$ is **primitive** if $c(f) = 1$.

More generally, for $f \in R[x]$ where $R$ is a UFD, $c(f)$ is the GCD of the coefficients (defined up to associates).

---

## 6. Gauss's Lemma

**Lemma 7.11 (Gauss's Lemma).** If $R$ is a UFD and $f, g \in R[x]$ are primitive, then $fg$ is primitive.

Equivalently: $c(fg) = c(f) \cdot c(g)$ (up to associates).

*Proof.* Suppose $fg$ is not primitive. Let $p$ be an irreducible dividing all coefficients of $fg$. Reduce mod $p$: $\bar{f}\bar{g} = \bar{0}$ in $(R/pR)[x]$. Since $p$ is prime in $R$ (UFD!), $R/pR$ is a domain, hence $(R/pR)[x]$ is a domain. So $\bar{f} = 0$ or $\bar{g} = 0$, contradicting primitivity of $f$ or $g$. $\blacksquare$

---

## 7. Gauss's Theorem: $R$ UFD $\implies$ $R[x]$ UFD

**Theorem 7.12 (Gauss).** If $R$ is a UFD, then $R[x]$ is a UFD.

*Proof outline.*

**Step 1.** Let $F = \text{Frac}(R)$. Since $F$ is a field, $F[x]$ is a UFD.

**Step 2.** Every $f \in R[x]$ can be written $f = c(f) \cdot f^*$ where $f^*$ is primitive and $c(f) \in R$.

**Step 3.** Show that irreducibles in $R[x]$ are either:
- (a) irreducible elements of $R$ (constants), or
- (b) primitive polynomials that are irreducible in $F[x]$.

**Step 4.** Since $R$ is a UFD, the content factors uniquely. Since $F[x]$ is a UFD, the primitive part factors uniquely. By Gauss's Lemma, products of primitives are primitive. So the whole factorisation is unique. $\blacksquare$

**Key Corollary.** $f \in \mathbb{Z}[x]$ primitive is irreducible over $\mathbb{Z}$ $\iff$ irreducible over $\mathbb{Q}$.

```
  Factorisation in ℤ[x]:
  
  f(x) = 6x³ + 9x² + 3x
  
  Step 1: Extract content
    c(f) = gcd(6,9,3) = 3
    f = 3 · (2x³ + 3x² + x) = 3 · x · (2x² + 3x + 1)
  
  Step 2: Factor primitive parts over ℚ (hence ℤ)
    2x² + 3x + 1 = (2x+1)(x+1)
  
  Step 3: Factor the content in ℤ
    3 is irreducible in ℤ
  
  Result: f(x) = 3 · x · (2x+1)(x+1)
```

---

## 8. Polynomial Rings in Several Variables

**Corollary.** If $R$ is a UFD, then $R[x_1, x_2, \ldots, x_n]$ is a UFD.

*Proof.* Induction: $R[x_1, \ldots, x_n] = R[x_1, \ldots, x_{n-1}][x_n]$. $\blacksquare$

Key examples:
- $\mathbb{Z}[x, y]$: UFD but not PID ($\langle x, y \rangle$ is not principal).
- $F[x, y]$: UFD but not PID.
- $\mathbb{Q}[x, y, z]$: UFD (multivariate polynomial ring over a field).

---

## 9. Irreducibles in $F[x]$

| Field $F$ | Irreducible polynomials |
|-----------|------------------------|
| $\mathbb{C}$ | Only linear (FTA) |
| $\mathbb{R}$ | Linear + irreducible quadratics |
| $\mathbb{Q}$ | Exist in every degree |
| $\mathbb{F}_p$ | About $p^n/n$ of degree $n$ |

---

## Summary Table

| Concept | Definition / Result |
|---------|-------------------|
| $R[x]$ | Polynomial ring with standard addition and multiplication |
| $\deg(fg)$ | $= \deg f + \deg g$ when $R$ is a domain |
| Division algorithm | Works in $F[x]$ for field $F$, not in $\mathbb{Z}[x]$ in general |
| Content $c(f)$ | GCD of coefficients |
| Primitive | $c(f) = 1$ |
| Gauss's Lemma | Primitive $\times$ primitive = primitive |
| Gauss's Theorem | $R$ UFD $\implies$ $R[x]$ UFD |
| Key corollary | Primitive $f \in \mathbb{Z}[x]$: irred over $\mathbb{Z}$ $\iff$ irred over $\mathbb{Q}$ |

---

## Quick Revision Questions

1. **Prove** that if $R$ is a domain, then $R[x]$ is a domain.

2. **State** Gauss's Lemma and outline its proof.

3. **Explain** why the division algorithm works in $\mathbb{Q}[x]$ but not in $\mathbb{Z}[x]$. Give a specific counterexample.

4. **Factor** $12x^3 - 6x^2 - 6x$ completely in $\mathbb{Z}[x]$.

5. **Prove** that $\mathbb{Z}[x, y]$ is a UFD but not a PID.

6. **State** the relationship between irreducibility of a primitive $f \in \mathbb{Z}[x]$ over $\mathbb{Z}$ and over $\mathbb{Q}$.

---

[← Previous: Euclidean Domains](03-euclidean-domains.md) | [Back to README](../README.md) | [Next: Irreducibility Criteria →](05-irreducibility-criteria.md)
