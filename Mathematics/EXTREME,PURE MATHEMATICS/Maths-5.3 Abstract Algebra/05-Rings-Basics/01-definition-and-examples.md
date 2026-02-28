# Chapter 5.1 — Rings: Definition and Examples

[← Previous: Applications](../04-Groups-Actions-Sylow/05-applications.md) | [Back to README](../README.md) | [Next: Subrings →](02-subrings.md)

---

## Overview

A **ring** is an algebraic structure with two operations—addition and multiplication—that generalises the integers. Rings arise naturally in number theory, geometry, and linear algebra. This chapter defines rings, explores their basic properties, and provides a gallery of key examples.

---

## 1. Definition

**Definition.** A **ring** $(R, +, \cdot)$ is a set $R$ equipped with two binary operations satisfying:

| Axiom | Statement |
|-------|-----------|
| (R1) $(R, +)$ is an abelian group | $a + b = b + a$; identity $0$; inverses $-a$ |
| (R2) Multiplication is associative | $a(bc) = (ab)c$ |
| (R3) Distributive laws hold | $a(b+c) = ab + ac$ and $(a+b)c = ac + bc$ |

```
  Ring axioms:
  
  ┌──────────────────────────────────────────────┐
  │  (R, +) is an abelian group                  │
  │  · is associative                            │
  │  · distributes over +  (both sides)          │
  └──────────────────────────────────────────────┘
  
  Optional properties:
  ┌──────────────────────────────────────────────┐
  │  Commutative:  ab = ba                       │
  │  Unital:       ∃ 1 ∈ R with 1·a = a·1 = a   │
  └──────────────────────────────────────────────┘
```

**Convention.** In this course, "ring" means **ring with unity** (multiplicative identity $1$ exists) unless stated otherwise. A ring is **commutative** if $ab = ba$ for all $a, b$.

---

## 2. Basic Properties

**Proposition 5.1.** In any ring $R$:

1. $0 \cdot a = a \cdot 0 = 0$ for all $a$
2. $(-a)b = a(-b) = -(ab)$
3. $(-a)(-b) = ab$
4. $a(b - c) = ab - ac$

*Proof of (1).* $0 \cdot a = (0 + 0) \cdot a = 0 \cdot a + 0 \cdot a$. Cancel $0 \cdot a$ from both sides: $0 = 0 \cdot a$. $\blacksquare$

---

## 3. Gallery of Examples

### 3.1. The Integers $\mathbb{Z}$

The prototypical commutative ring with unity. Not a field (no multiplicative inverses for most elements).

### 3.2. $\mathbb{Z}_n$ — Integers Modulo $n$

$\mathbb{Z}_n = \{0, 1, \ldots, n-1\}$ with addition and multiplication mod $n$.

Cayley table for $(\mathbb{Z}_4, \cdot)$:

| $\cdot$ | 0 | 1 | 2 | 3 |
|---------|---|---|---|---|
| **0** | 0 | 0 | 0 | 0 |
| **1** | 0 | 1 | 2 | 3 |
| **2** | 0 | 2 | 0 | 2 |
| **3** | 0 | 3 | 2 | 1 |

Note: $2 \cdot 2 = 0$ — so 2 is a **zero divisor** in $\mathbb{Z}_4$.

### 3.3. Matrix Rings $M_n(R)$

$n \times n$ matrices over a ring $R$. **Non-commutative** for $n \geq 2$.

Unity: the identity matrix $I_n$.

### 3.4. Polynomial Rings $R[x]$

Polynomials with coefficients in $R$. Commutative if $R$ is.

$$R[x] = \{a_0 + a_1 x + \cdots + a_n x^n : a_i \in R, n \geq 0\}$$

### 3.5. Gaussian Integers $\mathbb{Z}[i]$

$\mathbb{Z}[i] = \{a + bi : a, b \in \mathbb{Z}\}$ where $i^2 = -1$.

Commutative ring with unity. Actually an integral domain (and a Euclidean domain).

### 3.6. Function Rings

$\text{Fun}(X, R) = \{f : X \to R\}$ with pointwise operations: $(f+g)(x) = f(x) + g(x)$, $(fg)(x) = f(x)g(x)$.

### 3.7. Quaternions $\mathbb{H}$

$\mathbb{H} = \{a + bi + cj + dk : a,b,c,d \in \mathbb{R}\}$ with:

$$i^2 = j^2 = k^2 = ijk = -1$$

Non-commutative ring (actually a division ring / skew field).

### 3.8. Endomorphism Ring

$\text{End}(A)$ = set of group homomorphisms $A \to A$, with composition as multiplication.

---

## 4. Types of Rings

```
  Ring Hierarchy:
  
  Ring
   │
   ├── Commutative ring
   │    │
   │    ├── Integral domain  (no zero divisors)
   │    │    │
   │    │    ├── Unique factorization domain (UFD)
   │    │    │    │
   │    │    │    ├── Principal ideal domain (PID)
   │    │    │    │    │
   │    │    │    │    └── Euclidean domain (ED)
   │    │    │    │         │
   │    │    │    │         └── Field
   │    │    │    │
   │    │    │    └── (e.g., Z[x])
   │    │    │
   │    │    └── (e.g., Z[√-5])
   │    │
   │    └── (e.g., Z₆ — has zero divisors)
   │
   └── Non-commutative ring
        │
        └── Division ring (skew field)
             │
             └── (e.g., ℍ quaternions)
```

---

## 5. Units and Zero Divisors

**Definition.** An element $a \in R$ is:
- A **unit** if $\exists b \in R$ with $ab = ba = 1$. The set of units is $R^\times$ (a group under multiplication).
- A **zero divisor** if $a \neq 0$ and $\exists b \neq 0$ with $ab = 0$ or $ba = 0$.
- **Nilpotent** if $a^n = 0$ for some $n \geq 1$.
- **Idempotent** if $a^2 = a$.

### Examples in $\mathbb{Z}_{12}$:

| Element | Unit? | Zero divisor? | Nilpotent? | Idempotent? |
|---------|-------|---------------|-----------|-------------|
| 0 | No | — | Yes ($0^1=0$) | Yes |
| 1 | Yes ($1^{-1}=1$) | No | No | Yes |
| 2 | No | Yes ($2 \cdot 6 = 0$) | No | No |
| 3 | No | Yes ($3 \cdot 4 = 0$) | No | No |
| 4 | No | Yes ($4 \cdot 3 = 0$) | No | Yes |
| 5 | Yes ($5^{-1}=5$) | No | No | No |
| 6 | No | Yes ($6 \cdot 2 = 0$) | Yes ($6^2=0$) | No |
| 7 | Yes ($7^{-1}=7$) | No | No | No |
| 9 | No | Yes ($9 \cdot 4 = 0$) | No | Yes |
| 11 | Yes ($11^{-1}=11$) | No | No | No |

**Key fact:** $\mathbb{Z}_n^\times = \{a \in \mathbb{Z}_n : \gcd(a, n) = 1\}$, $|\mathbb{Z}_n^\times| = \varphi(n)$.

---

## 6. Trivial Ring and Zero Ring

- **Trivial ring:** $R = \{0\}$ with $0 + 0 = 0$, $0 \cdot 0 = 0$. Here $1 = 0$.
- A ring with $1 = 0$ is the trivial ring (since $a = 1 \cdot a = 0 \cdot a = 0$ for all $a$).

---

## 7. Comparison: Groups vs Rings vs Fields

| Property | Group | Ring | Field |
|----------|-------|------|-------|
| Operations | 1 | 2 | 2 |
| $(R, +)$ | — | Abelian group | Abelian group |
| $(R \setminus \{0\}, \cdot)$ | — | Monoid | Abelian group |
| Commutativity ($\cdot$) | Depends | Not required | Required |
| Division possible | — | Not always | Always ($\neq 0$) |
| Examples | $S_3, D_4$ | $\mathbb{Z}, M_2(\mathbb{R})$ | $\mathbb{Q}, \mathbb{F}_p$ |

---

## 8. Real-World Applications

| Application | Ring |
|------------|------|
| Modular arithmetic / Cryptography | $\mathbb{Z}_n$ |
| Error-correcting codes | $\mathbb{F}_q[x]/(f(x))$ |
| Computer graphics | $\mathbb{H}$ (quaternions for rotations) |
| Algebraic geometry | Coordinate rings $k[x,y]/(f)$ |
| Number theory | $\mathbb{Z}[\sqrt{d}]$, $\mathbb{Z}[\zeta_n]$ |

---

## Summary Table

| Concept | Definition / Key Fact |
|---------|----------------------|
| Ring | $(R,+,\cdot)$: abelian group $(R,+)$; associative $\cdot$; distributive |
| Commutative ring | $ab = ba$ for all $a, b$ |
| Unity (identity) | $\exists 1: 1a = a1 = a$ |
| Unit | Invertible element; units form group $R^\times$ |
| Zero divisor | $a \neq 0$, $\exists b \neq 0$: $ab = 0$ |
| Nilpotent | $a^n = 0$ for some $n$ |
| $\mathbb{Z}_n^\times$ | $\{a : \gcd(a,n)=1\}$, $\|\mathbb{Z}_n^\times\| = \varphi(n)$ |

---

## Quick Revision Questions

1. **Verify** that $\mathbb{Z}[i]$ satisfies all ring axioms.

2. **Find** all units and zero divisors in $\mathbb{Z}_{10}$.

3. **Prove** that $0 \cdot a = 0$ in any ring.

4. **Show** that $M_2(\mathbb{R})$ is non-commutative by giving a counterexample.

5. **Determine** whether $2\mathbb{Z}$ (even integers) is a ring with unity.

6. **Identify** the nilpotent elements in $\mathbb{Z}_{12}$.

---

[← Previous: Applications](../04-Groups-Actions-Sylow/05-applications.md) | [Back to README](../README.md) | [Next: Subrings →](02-subrings.md)
