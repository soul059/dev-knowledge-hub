# Chapter 7.3 — Euclidean Domains

[← Previous: Unique Factorisation Domains](02-unique-factorization-domains.md) | [Back to README](../README.md) | [Next: Polynomial Rings →](04-polynomial-rings.md)

---

## Overview

A **Euclidean domain** is an integral domain equipped with a division algorithm. This structure implies PID, which in turn implies UFD. We study the definition, prove the key theorem ED $\implies$ PID, and survey important examples.

---

## 1. Definition

**Definition.** An integral domain $R$ is a **Euclidean domain** (ED) if there exists a function

$$\delta: R \setminus \{0\} \to \mathbb{Z}_{\geq 0}$$

called a **Euclidean function** (or **Euclidean norm**), such that for any $a, b \in R$ with $b \neq 0$:

$$a = bq + r \quad \text{with } r = 0 \text{ or } \delta(r) < \delta(b)$$

for some $q, r \in R$.

Key point: we do not require $\delta(ab) = \delta(a)\delta(b)$ — multiplicativity is a bonus, not a requirement.

---

## 2. The Main Theorem: ED $\implies$ PID

**Theorem 7.5.** Every Euclidean domain is a principal ideal domain.

*Proof.* Let $I$ be a non-zero ideal in $R$. Choose $d \in I \setminus \{0\}$ with $\delta(d)$ minimal. We claim $I = (d)$.

$(d) \subseteq I$: clear since $d \in I$.

$I \subseteq (d)$: Let $a \in I$. By the division algorithm, $a = dq + r$ with $r = 0$ or $\delta(r) < \delta(d)$. Since $r = a - dq \in I$, and $\delta(d)$ is minimal among non-zero elements of $I$, we must have $r = 0$. So $a = dq \in (d)$. $\blacksquare$

**Corollary.** ED $\implies$ PID $\implies$ UFD.

---

## 3. Hierarchy of Ring Types

```
  ┌─────────────┐
  │   Field      │   every non-zero element is a unit
  └──────┬───────┘
         │
  ┌──────▼───────┐
  │  Euclidean   │   has a division algorithm
  │   Domain     │   examples: ℤ, F[x], ℤ[i], ℤ[ω]
  └──────┬───────┘
         │
  ┌──────▼───────┐
  │  Principal   │   every ideal is principal
  │  Ideal Domain│   example: ℤ[(1+√-19)/2]
  └──────┬───────┘
         │
  ┌──────▼───────┐
  │   Unique     │   unique factorisation into irreducibles
  │  Factor. Dom.│   example: ℤ[x], F[x,y]
  └──────┬───────┘
         │
  ┌──────▼───────┐
  │  Integral    │   no zero divisors
  │   Domain     │   example: ℤ[√-5]
  └──────────────┘
```

**All inclusions are strict.** See the examples below.

---

## 4. Standard Examples

### 4.1 $\mathbb{Z}$ with $\delta(n) = |n|$

The ordinary division algorithm: $a = bq + r$ with $0 \leq r < |b|$.

### 4.2 $F[x]$ with $\delta(f) = \deg(f)$

For any field $F$: polynomial long division gives $f = gq + r$ with $\deg(r) < \deg(g)$.

### 4.3 $\mathbb{Z}[i]$ (Gaussian Integers) with $\delta(\alpha) = N(\alpha) = a^2 + b^2$

**Theorem 7.6.** $\mathbb{Z}[i]$ is a Euclidean domain.

*Proof sketch.* Given $\alpha, \beta \in \mathbb{Z}[i]$ with $\beta \neq 0$, compute $\alpha/\beta \in \mathbb{Q}(i)$, round to the nearest Gaussian integer $q$. Then $r = \alpha - q\beta$ satisfies $N(r) < N(\beta)$ because the error in rounding has $N(\text{error}) \leq 1/4 + 1/4 = 1/2 < 1$. $\blacksquare$

```
  Division in ℤ[i]:
  
  α = (7 + 2i), β = (1 + i)
  
  α/β = (7+2i)(1-i)/|1+i|² = (7+2i)(1-i)/2 = (9-5i)/2 = 4.5 - 2.5i
  
  Round: q = 5 - 3i  (or 4 - 2i, etc.)
  Take q = 4 - 2i:
  r = (7+2i) - (4-2i)(1+i) = (7+2i) - (6-2i+4i-2i²) = (7+2i) - (6+2i+2) = (7+2i) - (8+0i)
  Hmm, let me recompute: (4-2i)(1+i) = 4+4i-2i-2i² = 4+2i+2 = 6+2i
  r = (7+2i)-(6+2i) = 1
  N(r) = 1 < N(1+i) = 2. ✓
```

### 4.4 $\mathbb{Z}[\omega]$ (Eisenstein Integers) with $\omega = e^{2\pi i/3} = \frac{-1+\sqrt{-3}}{2}$

Euclidean domain with norm $N(a + b\omega) = a^2 - ab + b^2$.

Used in Kummer's proof of Fermat's Last Theorem for regular primes.

### 4.5 $\mathbb{Z}[\sqrt{-2}]$ with $\delta(\alpha) = N(\alpha) = a^2 + 2b^2$

Euclidean domain — the rounding argument still works because $1/4 + 2 \cdot 1/4 = 3/4 < 1$.

---

## 5. Non-Euclidean PIDs

**Theorem (Motzkin, 1949).** $R = \mathbb{Z}\left[\frac{1+\sqrt{-19}}{2}\right]$ is a PID but not a Euclidean domain.

This shows ED $\subsetneq$ PID.

*Idea of proof.* One shows that $R$ is a Dedekind domain with class number 1 (hence PID), but no Euclidean function exists by universal side divisor arguments.

**Other examples:** $\mathbb{Z}\left[\frac{1+\sqrt{-43}}{2}\right]$, $\mathbb{Z}\left[\frac{1+\sqrt{-67}}{2}\right]$, $\mathbb{Z}\left[\frac{1+\sqrt{-163}}{2}\right]$ are all PIDs but not EDs.

These four correspond to the Heegner numbers $-19, -43, -67, -163$ (the "Baker–Heegner–Stark" list).

---

## 6. Euclidean Algorithm for GCD

In any Euclidean domain, the **Euclidean algorithm** computes $\gcd(a,b)$:

```
  gcd(a, b):
      while b ≠ 0:
          a, b = b, a mod b     (using the division algorithm)
      return a
```

**Example in $\mathbb{Z}$:** $\gcd(252, 198)$:

$$252 = 1 \cdot 198 + 54$$
$$198 = 3 \cdot 54 + 36$$
$$54 = 1 \cdot 36 + 18$$
$$36 = 2 \cdot 18 + 0$$

$\gcd(252, 198) = 18$.

**Extended Euclidean Algorithm** also gives Bézout coefficients: $d = ra + sb$.

---

## 7. Universal Property of ED

In a Euclidean domain:
- Bézout's identity: $\gcd(a,b) = ra + sb$
- Every ideal is principal (ED $\implies$ PID)
- Unique factorisation (PID $\implies$ UFD)
- Finite factorisation chains (PID $\implies$ ACCP)
- Irreducible $\iff$ prime

These make Euclidean domains the most computationally tractable class of rings.

---

## 8. Which Quadratic Rings are EDs?

**Theorem (Harper, Nagata, others).** Among rings $\mathbb{Z}[\sqrt{d}]$ (for square-free $d$):

For $d < 0$: Euclidean for $d = -1, -2$ only (with the standard norm). With $\mathbb{Z}\!\left[\frac{1+\sqrt{d}}{2}\right]$ (when $d \equiv 1 \pmod{4}$): also $d = -3, -7, -11$.

For $d > 0$: infinitely many Euclidean examples known ($d = 2, 3, 5, 6, 7, 11, 13, \ldots$).

| $d$ | Ring | ED? | PID? | UFD? |
|----|------|------|------|------|
| $-1$ | $\mathbb{Z}[i]$ | ✓ | ✓ | ✓ |
| $-2$ | $\mathbb{Z}[\sqrt{-2}]$ | ✓ | ✓ | ✓ |
| $-3$ | $\mathbb{Z}[\omega]$ | ✓ | ✓ | ✓ |
| $-5$ | $\mathbb{Z}[\sqrt{-5}]$ | ✗ | ✗ | ✗ |
| $-19$ | $\mathbb{Z}[\frac{1+\sqrt{-19}}{2}]$ | ✗ | ✓ | ✓ |
| $2$ | $\mathbb{Z}[\sqrt{2}]$ | ✓ | ✓ | ✓ |

---

## Summary Table

| Concept | Definition / Result |
|---------|-------------------|
| Euclidean function | $\delta: R\setminus\{0\} \to \mathbb{Z}_{\geq 0}$ allowing division with remainder |
| ED $\implies$ PID | Proved using minimal $\delta$-value in ideal |
| Hierarchy | Field $\subset$ ED $\subset$ PID $\subset$ UFD $\subset$ Domain |
| $\mathbb{Z}[i]$ is ED | Using norm $N(a+bi) = a^2+b^2$ |
| Non-Euclidean PID | $\mathbb{Z}[\frac{1+\sqrt{-19}}{2}]$ |
| Euclidean algorithm | Computes GCD iteratively |

---

## Quick Revision Questions

1. **State** the definition of a Euclidean domain and give two examples.

2. **Prove** that every Euclidean domain is a PID.

3. **Perform** the Euclidean algorithm to find $\gcd(11 + 3i, 1 + 8i)$ in $\mathbb{Z}[i]$.

4. **Explain** why $\mathbb{Z}[\sqrt{-2}]$ is a Euclidean domain but $\mathbb{Z}[\sqrt{-3}]$ is not.

5. **State** an example of a PID that is not a Euclidean domain and give its significance.

6. **Use** the Euclidean algorithm to express $\gcd(78, 21)$ as a linear combination of 78 and 21.

---

[← Previous: Unique Factorisation Domains](02-unique-factorization-domains.md) | [Back to README](../README.md) | [Next: Polynomial Rings →](04-polynomial-rings.md)
