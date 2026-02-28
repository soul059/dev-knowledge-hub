# Chapter 5: Properties of Modulo

[← Previous: Modular Exponentiation](04-modular-exponentiation.md) | [Back to README](../README.md) | [Next: Linear Congruences →](06-linear-congruences.md)

---

## Overview

This chapter consolidates all modular arithmetic properties into a single reference, covering the algebraic structure of $\mathbb{Z}/M\mathbb{Z}$, common identities, and the rules that make modular arithmetic work.

---

## 5.1 Complete Property Reference

```
  ┌──────────────────────────────────────────────────────────┐
  │  PRESERVED OPERATIONS (congruences preserved):           │
  │                                                          │
  │  If a ≡ a' (mod M) and b ≡ b' (mod M):                 │
  │                                                          │
  │  ✓ (a + b) ≡ (a' + b')         mod M                   │
  │  ✓ (a - b) ≡ (a' - b')         mod M                   │
  │  ✓ (a × b) ≡ (a' × b')         mod M                   │
  │  ✓ a^n     ≡ (a')^n             mod M  (n ≥ 0)         │
  │                                                          │
  │  ✗ a / b   ≢ a' / b'           mod M  (in general)     │
  │    → Must use modular inverse                            │
  │  ✗ log, sqrt, etc. NO modular version                   │
  └──────────────────────────────────────────────────────────┘
```

---

## 5.2 Modular Arithmetic as a Ring

```
  (ℤ/Mℤ, +, ×) forms a commutative ring:

  Addition:
  ┌────────────────────────────────────────┐
  │ Closure:  a + b mod M ∈ {0,...,M-1}   │
  │ Assoc:    (a+b)+c = a+(b+c) mod M     │
  │ Identity: a + 0 = a mod M             │
  │ Inverse:  a + (M-a) = 0 mod M         │
  │ Commut:   a + b = b + a mod M         │
  └────────────────────────────────────────┘

  Multiplication:
  ┌────────────────────────────────────────┐
  │ Closure:  a × b mod M ∈ {0,...,M-1}   │
  │ Assoc:    (a×b)×c = a×(b×c) mod M     │
  │ Identity: a × 1 = a mod M             │
  │ Commut:   a × b = b × a mod M         │
  │ Distrib:  a×(b+c) = a×b + a×c mod M   │
  └────────────────────────────────────────┘

  If M is PRIME → it becomes a FIELD!
  (every non-zero element has a multiplicative inverse)
```

---

## 5.3 Distributive Law Applications

```
  (a + b) × c mod M = (a×c + b×c) mod M

  Example: (3 + 4) × 5 mod 7
  Direct: 7 × 5 = 35 mod 7 = 0
  Distributed: 15 + 20 = 35 mod 7 = 0 ✓

  ┌──────────────────────────────────────────────────────────┐
  │  This is WHY we can take mod at each step of a sum:     │
  │                                                          │
  │  sum = Σ(aᵢ) mod M                                      │
  │  = ((a₁ mod M) + (a₂ mod M) + ... ) mod M              │
  │                                                          │
  │  Each intermediate result can be modded too!             │
  │  This prevents overflow.                                 │
  └──────────────────────────────────────────────────────────┘
```

---

## 5.4 Cancellation Laws

```
  ┌──────────────────────────────────────────────────────────┐
  │  ADDITIVE CANCELLATION:                                  │
  │  a + c ≡ b + c (mod M)  ⟹  a ≡ b (mod M)             │
  │  Always works!                                           │
  │                                                          │
  │  MULTIPLICATIVE CANCELLATION:                            │
  │  a×c ≡ b×c (mod M)  ⟹  a ≡ b (mod M/gcd(c,M))       │
  │                                                          │
  │  ⚠️ Only directly cancelable if gcd(c, M) = 1!         │
  │                                                          │
  │  Example: 6 ≡ 12 (mod 6)?  → 6 mod 6 = 0, 12 mod 6 = 0│
  │  Divide by 3: 2 ≡ 4 (mod 6)? → 2 ≠ 4 mod 6. WRONG!    │
  │  Correct: 2 ≡ 4 (mod 6/gcd(3,6)) = mod 2 → both 0. ✓  │
  └──────────────────────────────────────────────────────────┘
```

---

## 5.5 Modular Arithmetic with Common Moduli

```
  Special cases that appear frequently:

  mod 2:  → parity (even/odd)
  mod 3:  → digit sum divisibility
  mod 9:  → digital root
  mod 10: → last digit
  mod 10^k: → last k digits

  ┌────────┬────────────────────────────────────┐
  │ mod 2  │ x&1 (bitwise AND with 1)           │
  │ mod 4  │ x&3                                │
  │ mod 8  │ x&7                                │
  │ mod 2ⁿ │ x & (2ⁿ - 1)   (bitmask!)         │
  └────────┴────────────────────────────────────┘

  These bitmask tricks are MUCH faster than x % M!
```

---

## 5.6 Chinese Remainder Theorem (Preview)

```
  If gcd(M₁, M₂) = 1, then the system:
    x ≡ a₁ (mod M₁)
    x ≡ a₂ (mod M₂)
  
  has a UNIQUE solution modulo M₁ × M₂.

  Example:
    x ≡ 2 (mod 3)
    x ≡ 3 (mod 5)
  
  x = 8 (verify: 8%3=2 ✓, 8%5=3 ✓)
  Solution is unique mod 15: x ≡ 8 (mod 15)

  → Full treatment in Unit 10, Chapter 2.
```

---

## 5.7 Common Competition Patterns

### Pattern 1: Mod at Every Step

```
  // Computing sum of squares mod M
  result = 0
  FOR i = 1 TO n:
      result = (result + (i * i) % M) % M
  
  ⚠️ Even i*i should be computed with long long
  if i can be up to 10⁹!
```

### Pattern 2: Mod Preserving Equality

```
  If f(x) mod M = g(x) mod M, does f(x) = g(x)?
  
  NO! Only f(x) ≡ g(x) (mod M).
  They could differ by a multiple of M.
  
  This is why hash collisions occur!
```

### Pattern 3: Building Answer Digit by Digit

```
  For "last k digits of X":
  Compute X mod 10^k
  
  For "sum of digits":
  Cannot use modular arithmetic! Must process digits.
```

---

## 5.8 Modular Arithmetic Decision Tree

```
               ┌─ Need result mod M?
               │
     ┌─────────┴─────────┐
     │ Yes               │ No
     │                   │
  ┌──┴──┐               └─ Use normal arithmetic
  │     │
  Add/  Multiply/
  Sub   Power
  │     │
  Take  Take mod at
  mod   EACH step,
  at    use long long
  each  
  step  ┌── Division?
        │
     ┌──┴──┐
     │ Yes │ No → just mod
     │     │
  Find modular    
  inverse first!  
     │            
  ┌──┴──┐        
  │     │        
  M is  M not    
  prime prime    
  │     │        
  Fermat Extended 
  a^(p-2) Euclid 
```

---

## 5.9 Overflow Prevention Checklist

```
  ┌──────────────────────────────────────────────────────────┐
  │  CHECKLIST for modular arithmetic code:                  │
  │                                                          │
  │  □ Using 64-bit integers for all computations           │
  │  □ Taking mod after EVERY addition/multiplication       │
  │  □ Using ((x % M) + M) % M for subtraction             │
  │  □ Dividing FIRST when computing LCM                    │
  │  □ Using modular inverse instead of division            │
  │  □ base %= M at start of modPow                         │
  │  □ For M ≈ 10¹⁸, using __int128 or binary multiply     │
  │  □ Never computing full value then taking mod           │
  └──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Property | Formula | Always Works? |
|----------|---------|---------------|
| Addition | (a+b) mod M = ((a%M)+(b%M))%M | Yes |
| Subtraction | (a-b) mod M = ((a%M)-(b%M)+M)%M | Yes |
| Multiplication | (a×b) mod M = ((a%M)×(b%M))%M | Yes |
| Power | a^n mod M = modPow(a%M, n, M) | Yes |
| Division | (a/b) mod M = a×b⁻¹ mod M | When gcd(b,M)=1 |
| Cancellation | ac ≡ bc → a ≡ b (mod M/gcd(c,M)) | Adjusted mod |
| Ring structure | ℤ/Mℤ is a ring | Always |
| Field structure | ℤ/pℤ is a field | Only if p is prime |

---

## Quick Revision Questions

1. **What algebraic structure** does ℤ/Mℤ form? When is it a field?
2. **Can you cancel** 3 from both sides of 3a ≡ 3b (mod 9)?
3. **Why is x & (2ⁿ-1)** equivalent to x mod 2ⁿ?
4. **List three operations** that DON'T have modular equivalents.
5. **Write a checklist** to prevent overflow in modular arithmetic code.
6. **How does the Chinese Remainder Theorem** combine two modular equations?

---

[← Previous: Modular Exponentiation](04-modular-exponentiation.md) | [Back to README](../README.md) | [Next: Linear Congruences →](06-linear-congruences.md)
