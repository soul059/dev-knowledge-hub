# Chapter 7.2 — Unique Factorisation Domains

[← Previous: Divisibility in Integral Domains](01-divisibility-in-integral-domains.md) | [Back to README](../README.md) | [Next: Euclidean Domains →](03-euclidean-domains.md)

---

## Overview

A **unique factorisation domain** (UFD) is an integral domain where every non-zero, non-unit element factors uniquely into irreducibles — the natural generalisation of the Fundamental Theorem of Arithmetic. We study the definition, key characterisations, and examples of rings that are and are not UFDs.

---

## 1. Definition

**Definition.** An integral domain $R$ is a **unique factorisation domain** (UFD) if every non-zero, non-unit $a \in R$ can be written as:

$$a = p_1 p_2 \cdots p_n$$

where each $p_i$ is irreducible, and this factorisation is **unique** up to:
- reordering the factors, and
- replacing each $p_i$ by an associate $u_i p_i$.

Equivalently:
1. **Existence:** every non-zero, non-unit factors into irreducibles.
2. **Uniqueness:** if $p_1 \cdots p_m = q_1 \cdots q_n$ with all $p_i, q_j$ irreducible, then $m = n$ and (after reordering) $p_i \sim q_i$ for all $i$.

---

## 2. Key Characterisation

**Theorem 7.2.** In an integral domain $R$, the following are equivalent:
1. $R$ is a UFD.
2. Every non-zero, non-unit factors into irreducibles (existence), and every irreducible is prime.

This is the most useful characterisation: **irreducible $\iff$ prime in a UFD**.

*Proof sketch.*
- $(1 \Rightarrow 2)$: Let $p$ be irreducible and $p \mid ab$. Write $a, b$ in irreducible factorisations. Then $p$ times something equals the product $ab$, and by uniqueness $p$ must be associate to some factor of $a$ or $b$, giving $p \mid a$ or $p \mid b$.
- $(2 \Rightarrow 1)$: Since irreducibles are prime, the standard induction on length of factorisation gives uniqueness. $\blacksquare$

---

## 3. The Hierarchy

```
  Field ⊂ ED ⊂ PID ⊂ UFD ⊂ Integral Domain
  
  Examples:
  ℚ           ⊂    ℤ      ⊂     ℤ      ⊂     ℤ      ⊂     ℤ
  (field)     (ED)    (PID)   (UFD)   (domain)
  
  ℤ[x]  is a UFD but NOT a PID:
    (2, x) is not principal in ℤ[x]
  
  ℤ[√-5] is a domain but NOT a UFD:
    6 = 2·3 = (1+√-5)(1-√-5), two distinct factorisations
```

Strict containments:

| Containment | Example showing strictness |
|-------------|--------------------------|
| ED $\subsetneq$ PID | $\mathbb{Z}[\frac{1+\sqrt{-19}}{2}]$ is PID but not ED |
| PID $\subsetneq$ UFD | $\mathbb{Z}[x]$ is UFD but not PID |
| UFD $\subsetneq$ Domain | $\mathbb{Z}[\sqrt{-5}]$ is domain but not UFD |

---

## 4. Standard Examples of UFDs

### 4.1 $\mathbb{Z}$

The Fundamental Theorem of Arithmetic: every $n > 1$ factors uniquely into primes (up to sign). $\mathbb{Z}$ is a UFD (and also a PID and ED).

### 4.2 $F[x]$ for a field $F$

Every polynomial of degree $\geq 1$ factors uniquely into irreducible polynomials (up to scalar multiples). $F[x]$ is a UFD (also PID and ED).

### 4.3 $\mathbb{Z}[x]$

**Theorem (Gauss).** If $R$ is a UFD, then $R[x]$ is a UFD.

Since $\mathbb{Z}$ is a UFD, so is $\mathbb{Z}[x]$, and $\mathbb{Z}[x_1, \ldots, x_n]$, and $F[x_1, \ldots, x_n]$.

But $\mathbb{Z}[x]$ is **not** a PID: the ideal $(2, x)$ is not principal.

### 4.4 $\mathbb{Z}[i]$ (Gaussian Integers)

$\mathbb{Z}[i]$ is a Euclidean domain (hence UFD) with norm $N(a + bi) = a^2 + b^2$.

---

## 5. Non-UFD Examples

### 5.1 $\mathbb{Z}[\sqrt{-5}]$

$$6 = 2 \cdot 3 = (1 + \sqrt{-5})(1 - \sqrt{-5})$$

All four factors are irreducible (check norms: $N(2) = 4, N(3) = 9, N(1 \pm \sqrt{-5}) = 6$). But $2 \nsim (1 + \sqrt{-5})$ and $2 \nsim (1 - \sqrt{-5})$, so factorisations are genuinely different.

```
  Factorisation of 6 in ℤ[√-5]:
  
       6
      / \
     2   3         ← first factorisation
  
       6
      / \
  1+√-5  1-√-5    ← second factorisation
  
  No rearrangement/association makes these the same.
  ∴ ℤ[√-5] is not a UFD.
```

### 5.2 $\mathbb{Z}[\sqrt{-3}]$

$$4 = 2 \cdot 2 = (1 + \sqrt{-3})(1 - \sqrt{-3})$$

This ring also fails UFD. (However, $\mathbb{Z}[\omega]$ where $\omega = \frac{-1 + \sqrt{-3}}{2}$ is a PID.)

### 5.3 $\mathbb{Z}[2i]$

$$4 = 2 \cdot 2 = (2i)(-2i)$$

Not a UFD (and not even integrally closed).

---

## 6. The ACCP Condition

**Definition.** $R$ satisfies the **ascending chain condition on principal ideals** (ACCP) if every ascending chain

$$(a_1) \subseteq (a_2) \subseteq (a_3) \subseteq \cdots$$

stabilises: $\exists N$ such that $(a_n) = (a_N)$ for all $n \geq N$.

**Theorem 7.3.** If $R$ satisfies ACCP, then every non-zero non-unit has an irreducible factorisation (existence part of UFD).

*Proof.* If $a$ is not irreducible, $a = a_1 b_1$ with neither a unit. If $a_1$ not irreducible, $a_1 = a_2 b_2$, etc. This generates $(a) \subsetneq (a_1) \subsetneq (a_2) \subsetneq \cdots$, which must terminate by ACCP, giving an irreducible factor. Repeat to completely factor $a$. $\blacksquare$

**Theorem 7.4.** UFD $\implies$ ACCP. 

**Corollary.** $R$ is a UFD $\iff$ $R$ satisfies ACCP and every irreducible is prime.

---

## 7. Factorisation in Practice

**Strategy:** To factor $a \in R$ into irreducibles:

1. Compute $N(a)$ if a norm exists.
2. List irreducibles of small norm.
3. Trial divide.

**Example:** Factor $7 + i$ in $\mathbb{Z}[i]$.

$N(7+i) = 49 + 1 = 50 = 2 \cdot 5^2$.

Gaussian primes of norm $2$: $1+i$ (up to associates). $N(1+i) = 2$.

Divide: $(7+i)/(1+i) = (7+i)(1-i)/2 = (8-6i)/2 = 4-3i$.

$N(4-3i) = 16+9 = 25 = 5^2$.

Gaussian primes of norm $5$: $2+i$ (up to associates).

$(4-3i)/(2+i) = (4-3i)(2-i)/5 = (5-10i)/5 = 1-2i$.

$N(1-2i) = 5$. So $1-2i$ is irreducible.

$$7+i = (1+i)(2+i)(1-2i)$$

---

## 8. UFDs and Ideal Theory

| Property | In a UFD |
|----------|---------|
| Irreducible $\iff$ prime | Yes |
| GCD exists | Yes (take min exponents) |
| $\gcd(a,b) = ra + sb$? | Not always (only in PIDs) |
| Height-1 primes $=$ $(p)$ | Yes |
| Every ideal principal? | Not necessarily |

**Nagata's Theorem.** If $R$ is a domain satisfying ACCP, and $S$ is generated by primes of $R$, and $S^{-1}R$ is a UFD, then $R$ is a UFD.

---

## Summary Table

| Concept | Definition / Result |
|---------|-------------------|
| UFD | Existence + uniqueness of irreducible factorisation |
| Key characterisation | UFD $\iff$ existence of factorisations + irreducibles are prime |
| Hierarchy | Field $\subset$ ED $\subset$ PID $\subset$ UFD $\subset$ Domain |
| $R$ UFD $\implies$ $R[x]$ UFD | Gauss's theorem |
| $\mathbb{Z}[\sqrt{-5}]$ | Not a UFD: $6 = 2 \cdot 3 = (1+\sqrt{-5})(1-\sqrt{-5})$ |
| ACCP | Guarantees existence of factorisations |
| UFD $\iff$ | ACCP + every irreducible is prime |

---

## Quick Revision Questions

1. **State** the definition of a UFD and explain what "unique up to order and associates" means.

2. **Prove** that in a UFD, irreducible $\iff$ prime.

3. **Show** that $\mathbb{Z}[x]$ is a UFD but not a PID by considering the ideal $(2, x)$.

4. **Verify** that both factorisations $6 = 2 \cdot 3 = (1+\sqrt{-5})(1-\sqrt{-5})$ consist of irreducible elements in $\mathbb{Z}[\sqrt{-5}]$.

5. **Factor** $3 + 4i$ into Gaussian primes.

6. **State** the ACCP and explain its role in the theory of UFDs.

---

[← Previous: Divisibility in Integral Domains](01-divisibility-in-integral-domains.md) | [Back to README](../README.md) | [Next: Euclidean Domains →](03-euclidean-domains.md)
