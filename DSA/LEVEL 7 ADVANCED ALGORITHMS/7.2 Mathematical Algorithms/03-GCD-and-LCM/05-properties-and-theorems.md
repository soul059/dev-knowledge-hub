# Chapter 5: Properties and Theorems of GCD and LCM

[← Previous: GCD of Array](04-gcd-of-array.md) | [Back to README](../README.md) | [Next: Binary GCD →](06-binary-gcd.md)

---

## Overview

This chapter formalizes essential properties, identities, and theorems of GCD and LCM used extensively in number theory and competitive programming.

---

## 5.1 Fundamental GCD-LCM Identity

$$\gcd(a, b) \times \text{lcm}(a, b) = |a \times b|$$

### Proof via Prime Factorization

```
  Let a = p₁^α₁ × p₂^α₂ × ... × pₖ^αₖ
      b = p₁^β₁ × p₂^β₂ × ... × pₖ^βₖ
  (some exponents may be 0)

  GCD(a, b) = ∏ pᵢ^min(αᵢ, βᵢ)
  LCM(a, b) = ∏ pᵢ^max(αᵢ, βᵢ)

  GCD × LCM = ∏ pᵢ^(min(αᵢ,βᵢ) + max(αᵢ,βᵢ))

  Since: min(x, y) + max(x, y) = x + y  (always!)

  GCD × LCM = ∏ pᵢ^(αᵢ + βᵢ) = a × b   ■

  ⚠️ Does NOT extend to 3+ numbers:
  GCD(a,b,c) × LCM(a,b,c) ≠ a × b × c  in general!

  Example: a=2, b=3, c=6
  GCD=1, LCM=6, product=6
  But a×b×c = 36 ≠ 6
```

---

## 5.2 GCD Properties

```
  ┌──────────────────────────────────────────────────────┐
  │  ALGEBRAIC PROPERTIES:                                │
  │                                                      │
  │  1. Commutativity:  gcd(a, b) = gcd(b, a)           │
  │  2. Associativity:  gcd(a, gcd(b, c)) =             │
  │                     gcd(gcd(a, b), c)                │
  │  3. Identity:       gcd(a, 0) = |a|                 │
  │  4. Idempotent:     gcd(a, a) = |a|                 │
  │  5. Absorption:     gcd(a, lcm(a, b)) = |a|         │
  │  6. Distributive:   gcd(ka, kb) = k × gcd(a, b)     │
  │  7. Reduction:      gcd(a, b) = gcd(a, b mod a)     │
  │  8. Linearity:      gcd(a, b) | (xa + yb) ∀ x,y    │
  └──────────────────────────────────────────────────────┘
```

---

## 5.3 Coprimality (Relative Primes)

```
  a and b are COPRIME (relatively prime) ⟺ gcd(a, b) = 1

  ┌────────────────────────────────────────────────┐
  │  COPRIME PROPERTIES:                            │
  │                                                │
  │  If gcd(a, b) = 1:                             │
  │  1. ∃ x, y: ax + by = 1  (Bézout's identity)  │
  │  2. a has multiplicative inverse mod b          │
  │  3. lcm(a, b) = a × b                          │
  │  4. a | bc → a | c  (Euclid's lemma)           │
  │  5. a | n and b | n → ab | n                   │
  └────────────────────────────────────────────────┘

  Example pairs: (8, 15), (7, 12), (1, n), (p, q) where p, q distinct primes.
  
  Counter-example: gcd(8, 12) = 4 ≠ 1 → NOT coprime.
```

---

## 5.4 Euclid's Lemma

```
  THEOREM: If p is prime and p | ab, then p | a or p | b.

  More generally:
  If gcd(a, c) = 1 and a | bc, then a | c.

  Proof sketch:
  gcd(a, c) = 1 → ∃ x, y: ax + cy = 1
  Multiply by b: abx + bcy = b
  a | ab (trivially) and a | bc (given)
  So a | (abx + bcy) → a | b   ■

  This is the foundation of unique factorization!
```

---

## 5.5 Multiplicative Property

```
  ┌──────────────────────────────────────────────────────────┐
  │  GCD is a MULTIPLICATIVE function:                        │
  │                                                          │
  │  If gcd(a, b) = 1, then:                                │
  │    gcd(a×c, b) = gcd(c, b)                              │
  │                                                          │
  │  Proof:                                                  │
  │  Let d = gcd(ac, b)                                      │
  │  d | ac and d | b                                        │
  │  Let d₁ = gcd(d, a) → d₁ | a and d₁ | b → d₁ | gcd(a,b) = 1
  │  So d₁ = 1, meaning gcd(d, a) = 1                       │
  │  Since d | ac and gcd(d, a) = 1 → d | c (Euclid's lemma)│
  │  d | c and d | b → d | gcd(c, b)                        │
  │  Also gcd(c, b) | c and gcd(c, b) | b → gcd(c,b) | ac  │
  │  → gcd(c, b) | gcd(ac, b) = d                           │
  │  Therefore d = gcd(c, b)  ■                              │
  └──────────────────────────────────────────────────────────┘
```

---

## 5.6 GCD Chain Rules

```
  1. gcd(a, b) = gcd(a, b - a)     if b > a
  2. gcd(a, b) = gcd(a, b mod a)   (Euclidean algorithm)
  3. gcd(a, b, c) = gcd(gcd(a, b), c)

  4. gcd(a^m, b^m) = gcd(a, b)^m

     Proof: If a = ∏pᵢ^αᵢ, b = ∏pᵢ^βᵢ
     Then a^m = ∏pᵢ^(m·αᵢ), b^m = ∏pᵢ^(m·βᵢ)
     gcd(a^m, b^m) = ∏pᵢ^min(m·αᵢ, m·βᵢ)
                    = ∏pᵢ^(m·min(αᵢ, βᵢ))
                    = (∏pᵢ^min(αᵢ,βᵢ))^m = gcd(a,b)^m  ■

  5. gcd(Fₙ, Fₘ) = F_gcd(n,m)   (Fibonacci GCD identity!)
```

---

## 5.7 Fibonacci GCD Identity (Deep Dive)

```
  ┌──────────────────────────────────────────────────┐
  │  THEOREM: gcd(Fₙ, Fₘ) = F_{gcd(n, m)}          │
  │                                                  │
  │  Example: gcd(F₆, F₉)                           │
  │           = gcd(8, 34)                           │
  │           = 2                                    │
  │           = F₃                                   │
  │           = F_{gcd(6,9)} = F₃ ✓                  │
  │                                                  │
  │  Uses Fibonacci identity:                        │
  │  F_{m+n} = Fₘ₋₁Fₙ + FₘFₙ₊₁                     │
  │  → Fₘ | F_{km} for all k ≥ 1                    │
  │  (every k-th Fibonacci is divisible by Fₖ)       │
  └──────────────────────────────────────────────────┘

  Fibonacci:  1, 1, 2, 3, 5, 8, 13, 21, 34, ...
  Index:      1  2  3  4  5  6   7   8   9
```

---

## 5.8 Inclusion-Exclusion with GCD

```
  Count of integers in [1, n] coprime to m:

  φ(m) × (n / m) approximately, but exactly:

  Using Euler's Totient Function:
  Count = n × ∏(1 - 1/p) for all prime p dividing m

  Example: Count integers in [1, 30] coprime to 12.
  12 = 2² × 3
  Primes: 2, 3
  Count = 30 × (1 - 1/2) × (1 - 1/3) = 30 × 1/2 × 2/3 = 10
```

---

## 5.9 LCM Properties Revisited

```
  ┌──────────────────────────────────────────────────────────┐
  │  LATTICE STRUCTURE:                                       │
  │                                                          │
  │  GCD and LCM form a LATTICE on positive integers:       │
  │                                                          │
  │  1. Absorption laws:                                     │
  │     gcd(a, lcm(a, b)) = a                               │
  │     lcm(a, gcd(a, b)) = a                               │
  │                                                          │
  │  2. Distributive laws:                                   │
  │     gcd(a, lcm(b, c)) = lcm(gcd(a,b), gcd(a,c))       │
  │     lcm(a, gcd(b, c)) = gcd(lcm(a,b), lcm(a,c))       │
  │                                                          │
  │  3. Ordering:                                            │
  │     a | b  ⟺  gcd(a, b) = a  ⟺  lcm(a, b) = b       │
  └──────────────────────────────────────────────────────────┘
```

---

## 5.10 Application: The "Chicken McNugget" Theorem

```
  THEOREM (Sylvester-Frobenius for 2 values):
  If gcd(a, b) = 1, the largest number NOT expressible
  as xa + yb (x, y ≥ 0) is:

      ab - a - b

  Example: a = 3, b = 5
  Largest non-expressible = 15 - 3 - 5 = 7
  
  Verify:
  1 ✗  2 ✗  3=3  4 ✗  5=5  6=3+3  7 ✗  8=3+5  9=3+3+3  10=5+5
  11=3+3+5  12=3+3+3+3  ...all expressible from here!
  Largest gap = 7 ✓
```

---

## Summary Table

| Property | Formula | Condition |
|----------|---------|-----------|
| GCD-LCM identity | gcd×lcm = a×b | Two numbers only |
| Coprime LCM | lcm = a×b | gcd(a,b) = 1 |
| Power rule | gcd(aᵐ,bᵐ) = gcd(a,b)ᵐ | Always |
| Fibonacci GCD | gcd(Fₙ,Fₘ) = F_{gcd(n,m)} | Always |
| Distributive | gcd(a,lcm(b,c)) = lcm(gcd(a,b),gcd(a,c)) | Always |
| Absorption | gcd(a,lcm(a,b)) = a | Always |
| Euclid's lemma | gcd(a,c)=1, a\|bc → a\|c | Coprimality |

---

## Quick Revision Questions

1. **Prove** that min(x,y) + max(x,y) = x + y for all real x, y.
2. **Why does GCD × LCM = a × b** fail for three numbers? Give a counterexample.
3. **If gcd(a, b) = 1 and a | bc**, what can you conclude? State the theorem.
4. **Compute gcd(F₈, F₁₂)** using the Fibonacci GCD identity.
5. **What is the largest amount** you cannot make with coins of 7 and 11?
6. **Verify the absorption law**: gcd(12, lcm(12, 18)) = 12.

---

[← Previous: GCD of Array](04-gcd-of-array.md) | [Back to README](../README.md) | [Next: Binary GCD →](06-binary-gcd.md)
