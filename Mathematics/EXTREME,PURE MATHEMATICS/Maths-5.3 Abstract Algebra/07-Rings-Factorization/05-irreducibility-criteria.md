# Chapter 7.5 — Irreducibility Criteria

[← Previous: Polynomial Rings](04-polynomial-rings.md) | [Back to README](../README.md) | [Next: Field Extensions →](../08-Fields/01-field-extensions.md)

---

## Overview

Deciding whether a polynomial is irreducible over $\mathbb{Q}$ (or $\mathbb{Z}$, or $\mathbb{F}_p$) is a fundamental problem. This chapter presents the main **criteria** — Eisenstein, rational root test, reduction modulo $p$, and others — with worked examples.

---

## 1. Rational Root Test

**Theorem 7.13.** Let $f(x) = a_n x^n + \cdots + a_0 \in \mathbb{Z}[x]$. If $\frac{p}{q} \in \mathbb{Q}$ (in lowest terms) is a root of $f$, then:

$$p \mid a_0 \quad \text{and} \quad q \mid a_n$$

**Use:** If no rational root exists, $f$ has no linear factor. For degree 2 or 3, this implies irreducibility over $\mathbb{Q}$.

**Example:** $f(x) = x^3 + 3x + 1 \in \mathbb{Z}[x]$.

Possible rational roots: $\pm 1$. Check: $f(1) = 5 \neq 0$, $f(-1) = -3 \neq 0$. No rational root.

Since $\deg f = 3$, no rational root $\implies$ irreducible over $\mathbb{Q}$. $\checkmark$

**Warning:** For degree $\geq 4$, no rational root does NOT imply irreducibility. Example: $x^4 + 4 = (x^2 + 2x + 2)(x^2 - 2x + 2)$.

---

## 2. Eisenstein's Criterion

**Theorem 7.14 (Eisenstein).** Let $f(x) = a_n x^n + \cdots + a_0 \in \mathbb{Z}[x]$. If there exists a prime $p$ such that:

1. $p \nmid a_n$ (does not divide leading coefficient)
2. $p \mid a_i$ for all $0 \leq i < n$ (divides all other coefficients)
3. $p^2 \nmid a_0$ (does not divide constant term twice)

then $f$ is **irreducible over $\mathbb{Q}$**.

```
  Eisenstein Check for f(x) = aₙxⁿ + ... + a₁x + a₀ with prime p:
  
  Coefficient:   aₙ    aₙ₋₁   ...   a₁    a₀
                  │      │             │     │
  p divides?     NO    YES    ...    YES   YES
  p² divides?    ─      ─      ─      ─    NO
                 ▲                          ▲
             essential                  essential
```

*Proof.* Suppose $f = gh$ in $\mathbb{Z}[x]$ with $\deg g, \deg h \geq 1$. Reduce mod $p$: $\bar{f} = \bar{a}_n x^n$ in $\mathbb{F}_p[x]$. Since $\mathbb{F}_p[x]$ is a UFD, $\bar{g} = \bar{b}_s x^s$, $\bar{h} = \bar{c}_t x^t$ with $s + t = n$. So $p$ divides the constant terms of both $g$ and $h$, meaning $p^2 \mid a_0$. Contradiction. $\blacksquare$

---

## 3. Eisenstein: Worked Examples

**Example 1:** $f(x) = x^4 + 6x^3 + 9x^2 + 3x + 3$.

With $p = 3$: $3 \nmid 1$ (leading), $3 \mid 6, 9, 3, 3$, $9 \nmid 3$. Eisenstein applies. Irreducible over $\mathbb{Q}$.

**Example 2:** $\Phi_p(x) = \frac{x^p - 1}{x - 1} = x^{p-1} + x^{p-2} + \cdots + x + 1$ (the $p$-th cyclotomic polynomial for prime $p$).

Direct Eisenstein doesn't apply. Substitute $x = y + 1$:

$$\Phi_p(y+1) = \frac{(y+1)^p - 1}{y} = \sum_{k=1}^{p} \binom{p}{k} y^{k-1} = y^{p-1} + py^{p-2} + \cdots + \binom{p}{2}y + p$$

Now Eisenstein with $p$:
- Leading: $1$, not divisible by $p$. ✓
- Others: all $\binom{p}{k}$ divisible by $p$ for $1 \leq k \leq p-1$. ✓
- Constant: $p$, not divisible by $p^2$. ✓

So $\Phi_p(y+1)$ is irreducible, hence $\Phi_p(x)$ is irreducible over $\mathbb{Q}$.

**Example 3:** $f(x) = x^3 - 2$. With $p = 2$: $2 \nmid 1$, $2 \mid 0, 0, -2$, $4 \nmid -2$. Irreducible. ✓

---

## 4. Reduction Modulo $p$

**Theorem 7.15.** Let $f \in \mathbb{Z}[x]$ be primitive with $\deg f = n$. If for some prime $p$ with $p \nmid a_n$:

$$\bar{f} \in \mathbb{F}_p[x] \text{ is irreducible}$$

then $f$ is irreducible over $\mathbb{Q}$.

*Proof.* If $f = gh$ in $\mathbb{Z}[x]$ with $\deg g, \deg h \geq 1$, then $\bar{f} = \bar{g}\bar{h}$ in $\mathbb{F}_p[x]$ with $\deg \bar{g} = \deg g$, $\deg \bar{h} = \deg h$ (since $p \nmid a_n$). Contradicts $\bar{f}$ irreducible. $\blacksquare$

**Example:** $f(x) = x^4 + x + 1 \in \mathbb{Z}[x]$.

Reduce mod 2: $\bar{f} = x^4 + x + 1 \in \mathbb{F}_2[x]$. Check: $\bar{f}(0) = 1 \neq 0$, $\bar{f}(1) = 1 \neq 0$, so no roots. Not a product of two quadratics: the only irreducible quadratic over $\mathbb{F}_2$ is $x^2 + x + 1$, and $(x^2+x+1)^2 = x^4 + x^2 + 1 \neq x^4 + x + 1$. So $\bar{f}$ is irreducible over $\mathbb{F}_2$, hence $f$ is irreducible over $\mathbb{Q}$.

**Warning (converse fails).** $x^4 + 1$ is irreducible over $\mathbb{Q}$ but reducible mod every prime $p$.

```
  Reduction mod p strategy:
  
  f(x) ∈ ℤ[x]   ───mod p───▸   f̄(x) ∈ 𝔽ₚ[x]
       │                              │
       │  if f̄ irreducible            │
       │◄─────── then f irreducible ──┘
       │
       │  if f̄ reducible
       │  ⟹ NOTHING (try another p)
```

---

## 5. Degree Considerations

| Degree | Methods |
|--------|---------|
| 1 | Always irreducible (non-constant, primitive) |
| 2, 3 | Irreducible $\iff$ no roots (in the field) |
| $\geq 4$ | No-root is necessary but not sufficient; must check all possible factor-degree splits |

For degree 4: must check no roots AND not a product of two quadratics.

For degree 5: must check no roots, not degree 1 + 4, not degree 2 + 3.

---

## 6. Substitution Tricks

**Theorem 7.16.** If $f(x + c)$ is irreducible for some $c \in \mathbb{Z}$, then $f(x)$ is irreducible.

Also: if $f(ax)$ is irreducible and $a \neq 0$, then $f(x)$ is irreducible.

**Example:** $f(x) = x^4 + 1$.

Try $f(x+1) = x^4 + 4x^3 + 6x^2 + 4x + 2$. Eisenstein with $p = 2$: $2 \nmid 1$, $2 \mid 4, 6, 4, 2$, $4 \nmid 2$. Irreducible! ✓

---

## 7. Irreducibility over Finite Fields

Over $\mathbb{F}_p$, one can check irreducibility computationally:

**Theorem 7.17.** $f \in \mathbb{F}_p[x]$ of degree $n$ is irreducible $\iff$ $f \mid (x^{p^n} - x)$ and $\gcd(f, x^{p^k} - x) = 1$ for all $1 \leq k < n$ dividing $n$.

**Count:** The number of monic irreducible polynomials of degree $n$ over $\mathbb{F}_p$ is:

$$\frac{1}{n} \sum_{d \mid n} \mu(n/d) \, p^d$$

where $\mu$ is the Möbius function. (This is always $\geq 1$.)

| $p$ | Degree 1 | Degree 2 | Degree 3 | Degree 4 |
|-----|---------|---------|---------|---------|
| 2 | 2 | 1 | 2 | 3 |
| 3 | 3 | 3 | 8 | 18 |
| 5 | 5 | 10 | 40 | 150 |

---

## 8. Summary of Techniques

```
  Given f(x) ∈ ℤ[x], is f irreducible over ℚ?
  
  ┌─────────────────────────────────────────────┐
  │ 1. Extract content → make f primitive       │
  │ 2. If deg ≤ 3: rational root test           │
  │ 3. Try Eisenstein (possibly after x ↦ x+c)  │
  │ 4. Reduce mod p for small primes            │
  │ 5. Check specific factorisation patterns     │
  │ 6. Use Galois group / resolvent methods      │
  └─────────────────────────────────────────────┘
```

---

## Summary Table

| Criterion | Statement |
|-----------|----------|
| Rational Root Test | Roots $\frac{p}{q}$: $p \mid a_0$, $q \mid a_n$ |
| Eisenstein ($p$) | $p \nmid a_n$, $p \mid a_i$ ($i < n$), $p^2 \nmid a_0$ $\implies$ irreducible |
| Reduction mod $p$ | $\bar{f}$ irreducible in $\mathbb{F}_p[x]$ $\implies$ $f$ irreducible over $\mathbb{Q}$ |
| Substitution | $f(x+c)$ irreducible $\implies$ $f(x)$ irreducible |
| Degree 2, 3 | Irreducible $\iff$ no roots |
| Cyclotomic $\Phi_p$ | Irreducible via Eisenstein on $\Phi_p(x+1)$ |

---

## Quick Revision Questions

1. **Apply** Eisenstein's criterion (with appropriate prime $p$) to prove $x^5 + 15x^3 + 25x^2 + 5$ is irreducible over $\mathbb{Q}$.

2. **Prove** that $x^4 + x + 1$ is irreducible over $\mathbb{Q}$ by reduction modulo 2.

3. **Show** that $x^4 + 1$ is irreducible over $\mathbb{Q}$ using the substitution $x \mapsto x + 1$ and Eisenstein.

4. **Use** the rational root test to show $x^3 - 5x + 1$ is irreducible over $\mathbb{Q}$.

5. **State** the converse failure of mod-$p$ reduction: give an example of $f$ irreducible over $\mathbb{Q}$ but reducible over $\mathbb{F}_p$ for every $p$.

6. **Compute** the number of irreducible monic polynomials of degree 4 over $\mathbb{F}_3$.

---

[← Previous: Polynomial Rings](04-polynomial-rings.md) | [Back to README](../README.md) | [Next: Field Extensions →](../08-Fields/01-field-extensions.md)
