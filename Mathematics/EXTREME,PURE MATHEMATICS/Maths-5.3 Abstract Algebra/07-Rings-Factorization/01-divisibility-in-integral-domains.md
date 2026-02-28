# Chapter 7.1 — Divisibility in Integral Domains

[← Previous: Principal Ideal Domains](../06-Rings-Ideals/05-principal-ideal-domains.md) | [Back to README](../README.md) | [Next: Unique Factorisation Domains →](02-unique-factorization-domains.md)

---

## Overview

This chapter develops the theory of **divisibility** in general integral domains — notions of associates, irreducibles, and primes — generalising familiar properties from $\mathbb{Z}$. These concepts form the foundation for unique factorisation theory.

---

## 1. Divisibility

**Definition.** In an integral domain $R$, we say $a$ **divides** $b$ (written $a \mid b$) if $b = ac$ for some $c \in R$.

Equivalently: $(b) \subseteq (a)$.

| Property | Statement | Ideal form |
|----------|-----------|-----------|
| $a \mid b$ | $\exists c: b = ac$ | $(b) \subseteq (a)$ |
| $a \mid b$ and $b \mid a$ | $a$ and $b$ are associates | $(a) = (b)$ |
| $a \mid 1$ | $a$ is a unit | $(a) = R$ |

---

## 2. Associates

**Definition.** Elements $a, b \in R$ are **associates** (written $a \sim b$) if $a = ub$ for some unit $u \in R^\times$.

Equivalently: $a \mid b$ and $b \mid a$, i.e., $(a) = (b)$.

**Examples:**
- In $\mathbb{Z}$: $a \sim b \iff a = \pm b$. Associates: $\{3, -3\}$, $\{7, -7\}$.
- In $\mathbb{Z}[i]$: units are $\{\pm 1, \pm i\}$. Associates of $1 + i$: $\{1+i, -1-i, i-1, 1-i\} \cdot \ldots$ actually $\{1+i, -1-i, -1+i, 1-i\}$.
- In $F[x]$: units are $F^\times$ (non-zero constants). $2x + 2 \sim x + 1$ in $\mathbb{Q}[x]$.

---

## 3. Irreducible Elements

**Definition.** A non-zero, non-unit element $p \in R$ is **irreducible** if:

$$p = ab \implies a \text{ is a unit or } b \text{ is a unit}$$

"Cannot be factored non-trivially."

```
  Irreducible:  p ≠ 0, p ∉ R×, and
                p = ab ⟹ a ∈ R× or b ∈ R×
  
  In ℤ: irreducible = ±{prime numbers}
  In F[x]: irreducible = irreducible polynomials
  In ℤ[i]: irreducible? Depends on Gaussian integer norm.
```

---

## 4. Prime Elements

**Definition.** A non-zero, non-unit element $p \in R$ is **prime** if:

$$p \mid ab \implies p \mid a \text{ or } p \mid b$$

Equivalently: $(p)$ is a prime ideal.

---

## 5. Prime vs Irreducible

**Theorem 7.1.** In any integral domain:

$$\text{prime} \implies \text{irreducible}$$

*Proof.* Let $p$ be prime and $p = ab$. Then $p \mid ab$, so $p \mid a$ or $p \mid b$. Say $p \mid a$: $a = pc$. Then $p = ab = pcb$, so $1 = cb$ (cancel $p$), making $b$ a unit. $\blacksquare$

**The converse is false in general!**

### Counterexample: $\mathbb{Z}[\sqrt{-5}]$

In $R = \mathbb{Z}[\sqrt{-5}]$, consider $2$.

$2$ is **irreducible**: $N(2) = 4$. If $2 = \alpha\beta$, then $N(\alpha)N(\beta) = 4$. Since $N(a + b\sqrt{-5}) = a^2 + 5b^2$, the only values $\leq 4$ are $1$ and $4$ (no element has norm 2 or 3). So one factor must be a unit.

$2$ is **not prime**: $2 \mid 6 = (1 + \sqrt{-5})(1 - \sqrt{-5})$, but $2 \nmid (1 + \sqrt{-5})$ and $2 \nmid (1 - \sqrt{-5})$.

```
  In ℤ[√-5]:
  
  6 = 2 · 3 = (1+√-5)(1-√-5)
  
  Two genuinely different factorisations!
  
  2 is irreducible but NOT prime ⟹ unique factorisation fails
  
  prime ⟹ irreducible:  ALWAYS
  irreducible ⟹ prime:  Only in UFDs (and PIDs)
```

---

## 6. Norm Functions

A **norm** on $R$ is a function $N: R \to \mathbb{Z}_{\geq 0}$ satisfying:

1. $N(a) = 0 \iff a = 0$
2. $N(ab) = N(a)N(b)$ (multiplicativity)

Norms are powerful tools for proving irreducibility.

| Ring | Norm $N(a + b\sqrt{d})$ | Units ($N(u) = 1$) |
|------|------------------------|-------------------|
| $\mathbb{Z}[i]$ | $a^2 + b^2$ | $\{\pm 1, \pm i\}$ |
| $\mathbb{Z}[\sqrt{-2}]$ | $a^2 + 2b^2$ | $\{\pm 1\}$ |
| $\mathbb{Z}[\sqrt{-5}]$ | $a^2 + 5b^2$ | $\{\pm 1\}$ |
| $\mathbb{Z}[\sqrt{2}]$ | $|a^2 - 2b^2|$ | $\{\pm(1+\sqrt{2})^n\}$ (infinite!) |

**Irreducibility test:** If $N(p)$ is a rational prime, then $p$ is irreducible.

---

## 7. GCD and LCM

**Definition.** In an integral domain $R$:

- $d = \gcd(a, b)$ if $d \mid a$, $d \mid b$, and whenever $c \mid a, c \mid b$, then $c \mid d$.
- $m = \text{lcm}(a, b)$ if $a \mid m$, $b \mid m$, and whenever $a \mid c, b \mid c$, then $m \mid c$.

GCDs and LCMs need not exist in a general domain.

In a **UFD**, GCDs always exist (take min of exponents in prime factorisations).

In a **PID**, $\gcd(a,b) = d$ where $(d) = (a,b)$.

---

## 8. Comparison Table

| Property | Int. Domain | UFD | PID | ED | Field |
|----------|------------|-----|-----|-----|-------|
| No zero divisors | ✓ | ✓ | ✓ | ✓ | ✓ |
| Unique factorisation | ✗ | ✓ | ✓ | ✓ | ✓ |
| Every ideal principal | ✗ | ✗ | ✓ | ✓ | ✓ |
| Division algorithm | ✗ | ✗ | ✗ | ✓ | ✓ |
| Every non-zero invertible | ✗ | ✗ | ✗ | ✗ | ✓ |
| Irred $\iff$ prime | ✗ | ✓ | ✓ | ✓ | — |
| GCD exists | ✗ | ✓ | ✓ | ✓ | — |

---

## Summary Table

| Concept | Definition |
|---------|-----------|
| $a \mid b$ | $\exists c: b = ac$; equivalently $(b) \subseteq (a)$ |
| Associates | $a = ub$, $u$ a unit; $(a) = (b)$ |
| Irreducible | $p = ab \implies a$ or $b$ is a unit |
| Prime | $p \mid ab \implies p \mid a$ or $p \mid b$ |
| Prime $\implies$ irreducible | Always in domains |
| Irreducible $\implies$ prime | Only in UFDs |
| Norm | $N: R \to \mathbb{Z}_{\geq 0}$, multiplicative |

---

## Quick Revision Questions

1. **Prove** that prime $\implies$ irreducible in any integral domain.

2. **Show** that $3$ is irreducible but not prime in $\mathbb{Z}[\sqrt{-5}]$.

3. **Find** all associates of $2 + i$ in $\mathbb{Z}[i]$.

4. **Compute** $\gcd(8 + 6i, 5 + 3i)$ in $\mathbb{Z}[i]$ using the Euclidean algorithm.

5. **Explain** why $\mathbb{Z}[\sqrt{-5}]$ is not a UFD using the factorisation of 6.

6. **Determine** whether $1 + \sqrt{-2}$ is irreducible in $\mathbb{Z}[\sqrt{-2}]$.

---

[← Previous: Principal Ideal Domains](../06-Rings-Ideals/05-principal-ideal-domains.md) | [Back to README](../README.md) | [Next: Unique Factorisation Domains →](02-unique-factorization-domains.md)
