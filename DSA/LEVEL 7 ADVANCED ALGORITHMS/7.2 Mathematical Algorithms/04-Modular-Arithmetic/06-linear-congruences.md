# Chapter 6: Linear Congruences

[← Previous: Properties of Modulo](05-properties-of-modulo.md) | [Back to README](../README.md) | [Next: Unit 5 — Exponentiation by Squaring →](../05-Fast-Exponentiation/01-exponentiation-by-squaring.md)

---

## Overview

A **linear congruence** is an equation of the form $ax \equiv b \pmod{M}$. This chapter covers when solutions exist, how to find them, and their applications including the Chinese Remainder Theorem workflow.

---

## 6.1 Problem Statement

```
  Find x such that: ax ≡ b (mod M)

  Equivalently: find x where M | (ax - b)
  Or: ax - b = kM for some integer k
  Or: ax - kM = b  → Linear Diophantine equation!
```

---

## 6.2 Existence of Solutions

```
  ┌──────────────────────────────────────────────────────────┐
  │  THEOREM:                                                │
  │  ax ≡ b (mod M) has a solution                          │
  │  ⟺  gcd(a, M) | b                                     │
  │                                                          │
  │  If d = gcd(a, M) divides b:                            │
  │  → There are exactly d solutions modulo M               │
  │  → One solution every M/d apart                         │
  │                                                          │
  │  If d does not divide b:                                │
  │  → NO solution exists                                   │
  └──────────────────────────────────────────────────────────┘
```

### Examples

```
  1) 3x ≡ 6 (mod 9)
     d = gcd(3, 9) = 3
     3 | 6? Yes → solutions exist!
     Divide through by 3: x ≡ 2 (mod 3)
     Solutions mod 9: x ∈ {2, 5, 8}
     Check: 3×2=6≡6, 3×5=15≡6, 3×8=24≡6 ✓

  2) 3x ≡ 7 (mod 9)
     d = gcd(3, 9) = 3
     3 | 7? No → NO solution!

  3) 5x ≡ 1 (mod 7)
     d = gcd(5, 7) = 1
     1 | 1? Yes → exactly 1 solution
     x = 5⁻¹ mod 7 = 3 (since 5×3=15≡1 mod 7)
```

---

## 6.3 Solving Algorithm

```
FUNCTION solveLinearCongruence(a, b, M):
    d = gcd(a, M)
    IF b % d ≠ 0:
        RETURN "No solution"
    
    // Reduce: (a/d)x ≡ (b/d) (mod M/d)
    a' = a / d
    b' = b / d
    M' = M / d
    
    // Now gcd(a', M') = 1, so inverse exists
    x₀ = (b' × modInverse(a', M')) % M'
    
    // All d solutions
    solutions = []
    FOR i = 0 TO d - 1:
        solutions.add((x₀ + i × M') % M)
    
    RETURN solutions
```

---

## 6.4 Detailed Trace: 6x ≡ 4 (mod 10)

```
  a = 6, b = 4, M = 10
  d = gcd(6, 10) = 2
  2 | 4? Yes → solutions exist (exactly 2)

  Reduce: (6/2)x ≡ (4/2) (mod 10/2)
          3x ≡ 2 (mod 5)

  Find 3⁻¹ mod 5:
  3 × 1 = 3, 3 × 2 = 6 ≡ 1 mod 5
  So 3⁻¹ = 2

  x₀ = 2 × 2 mod 5 = 4

  All solutions mod 10:
  x₁ = 4
  x₂ = 4 + 5 = 9

  Solutions: {4, 9}

  Verify:
  6 × 4 = 24 mod 10 = 4 ✓
  6 × 9 = 54 mod 10 = 4 ✓
```

---

## 6.5 Systems of Congruences (CRT Setup)

```
  System:
    x ≡ a₁ (mod M₁)
    x ≡ a₂ (mod M₂)

  If gcd(M₁, M₂) = 1 (Chinese Remainder Theorem):
  
  x = a₁ × M₂ × (M₂⁻¹ mod M₁) + a₂ × M₁ × (M₁⁻¹ mod M₂)
  x mod (M₁ × M₂)

  Example:
    x ≡ 2 (mod 3)
    x ≡ 3 (mod 5)
  
  M₂⁻¹ mod M₁ = 5⁻¹ mod 3 = 2 (5×2=10≡1 mod 3)
  M₁⁻¹ mod M₂ = 3⁻¹ mod 5 = 2 (3×2=6≡1 mod 5)
  
  x = 2×5×2 + 3×3×2 = 20 + 18 = 38
  x mod 15 = 38 mod 15 = 8

  Verify: 8 mod 3 = 2 ✓, 8 mod 5 = 3 ✓
```

---

## 6.6 Generalized CRT (Non-coprime Moduli)

```
  If gcd(M₁, M₂) = d > 1:
  System has solution ⟺  a₁ ≡ a₂ (mod d)
  
  If solution exists:
  Combined modulus = lcm(M₁, M₂)

  Example:
    x ≡ 2 (mod 6)
    x ≡ 8 (mod 10)
  
  d = gcd(6, 10) = 2
  2 ≡ 8 (mod 2)? → 0 ≡ 0 ✓ → solution exists!
  
  Combined modulus = lcm(6, 10) = 30
  Find x: try x = 2, 8, 14, 20, 26 (≡2 mod 6)
  Which are ≡8 mod 10?  8 mod 10 = 8, 18 mod 10 = 8
  x = 8 works: 8%6=2 ✓, 8%10=8 ✓
  x ≡ 8 (mod 30)
```

---

## 6.7 Application: Scheduling Problems

```
  Problem: Bus A comes every 12 minutes (first at 7:00)
           Bus B comes every 18 minutes (first at 7:05)
           When do both arrive at the same time?

  Bus A at minute: 0, 12, 24, 36, 48, 60, ...
  Bus B at minute: 5, 23, 41, 59, ...

  Find t: t ≡ 0 (mod 12) and t ≡ 5 (mod 18)

  d = gcd(12, 18) = 6
  0 ≡ 5 (mod 6)? → 0 ≠ 5 mod 6 → NO SOLUTION!

  The buses NEVER arrive together (for this configuration).

  Modify: Bus B first at 7:06
  t ≡ 0 (mod 12) and t ≡ 6 (mod 18)
  d = 6, 0 ≡ 6 (mod 6)? → 0 ≡ 0 ✓
  t = 24: 24%12=0 ✓, 24%18=6 ✓
  Period = lcm(12, 18) = 36
  They meet at minutes: 24, 60, 96, ...
```

---

## 6.8 Linear Congruence as Matrix Problem

```
  The system:
    x ≡ a₁ (mod M₁)
    x ≡ a₂ (mod M₂)
    ...
    x ≡ aₖ (mod Mₖ)

  Can be solved incrementally:
  
  Start with x₀ = a₁, mod₀ = M₁
  For each new equation x ≡ aᵢ (mod Mᵢ):
    Merge with current (x₀, mod₀)
    Find new x₀ in [0, lcm(mod₀, Mᵢ))
    Update mod₀ = lcm(mod₀, Mᵢ)

  This "folding" approach handles any number of equations.
```

---

## 6.9 Common Problem Types

| Problem Type | Technique |
|-------------|-----------|
| Find x: ax ≡ b (mod M) | Linear congruence (this chapter) |
| Find x: x ≡ aᵢ (mod Mᵢ) for all i | CRT |
| Modular inverse | Special case: ax ≡ 1 (mod M) |
| Diophantine equation | ax + by = c (Extended GCD) |
| Simultaneous schedules | CRT with practical modeling |

---

## Summary Table

| Concept | Key Formula | Condition |
|---------|-------------|-----------|
| Solution exists | gcd(a,M) \| b | Necessary and sufficient |
| Number of solutions | d = gcd(a, M) | Exactly d solutions mod M |
| Solving method | Reduce, find inverse, enumerate | gcd(a', M') = 1 after reduction |
| CRT (coprime) | Unique solution mod M₁×M₂ | gcd(M₁, M₂) = 1 |
| CRT (general) | Solution mod lcm(M₁, M₂) | a₁ ≡ a₂ (mod gcd(M₁,M₂)) |

---

## Quick Revision Questions

1. **Does 4x ≡ 5 (mod 6)** have a solution? Why or why not?
2. **Solve 6x ≡ 3 (mod 9)** and list all solutions.
3. **Using CRT, solve** x ≡ 1 (mod 3) and x ≡ 2 (mod 5).
4. **Two events repeat** every 8 and 12 minutes. When do they coincide?
5. **How many solutions** does ax ≡ b (mod M) have when gcd(a, M) = d and d | b?
6. **Why is modular inverse** a special case of linear congruence?

---

[← Previous: Properties of Modulo](05-properties-of-modulo.md) | [Back to README](../README.md) | [Next: Unit 5 — Exponentiation by Squaring →](../05-Fast-Exponentiation/01-exponentiation-by-squaring.md)
