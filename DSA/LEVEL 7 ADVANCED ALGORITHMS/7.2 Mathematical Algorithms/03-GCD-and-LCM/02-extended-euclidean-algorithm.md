# Chapter 2: Extended Euclidean Algorithm

[← Previous: Euclidean Algorithm](01-euclidean-algorithm.md) | [Back to README](../README.md) | [Next: LCM Computation →](03-lcm-computation.md)

---

## Overview

The Extended Euclidean Algorithm not only computes GCD(a, b), but also finds integers x and y such that **ax + by = GCD(a, b)** (Bézout's identity). This is crucial for computing modular inverses, solving linear Diophantine equations, and cryptographic key generation.

---

## 2.1 Bézout's Identity

$$\text{For any integers } a, b, \text{ there exist integers } x, y \text{ such that: } ax + by = \gcd(a, b)$$

```
  ┌──────────────────────────────────────────────────────────┐
  │  BÉZOUT'S IDENTITY:                                      │
  │                                                          │
  │  ax + by = GCD(a, b)                                     │
  │                                                          │
  │  • x and y are called Bézout coefficients                │
  │  • They are NOT unique (infinitely many solutions)       │
  │  • The Extended GCD finds one particular solution        │
  │                                                          │
  │  Example: 35x + 15y = 5                                  │
  │  One solution: x = 1, y = -2  (35×1 + 15×(-2) = 5) ✓   │
  │  Another: x = -2, y = 5  (35×(-2) + 15×5 = 5) ✓         │
  └──────────────────────────────────────────────────────────┘
```

---

## 2.2 Algorithm (Pseudocode)

```
FUNCTION extendedGCD(a, b):
    IF b == 0:
        RETURN (a, 1, 0)    // GCD = a, x = 1, y = 0
                             // because a×1 + 0×0 = a
    
    (g, x₁, y₁) = extendedGCD(b, a % b)
    
    // Back-substitute:
    x = y₁
    y = x₁ - ⌊a/b⌋ × y₁
    
    RETURN (g, x, y)
```

### Why the Back-Substitution Works

```
  We know: b × x₁ + (a % b) × y₁ = g     ...(from recursive call)
  
  Since a % b = a - ⌊a/b⌋ × b:
  
  b × x₁ + (a - ⌊a/b⌋ × b) × y₁ = g
  b × x₁ + a × y₁ - ⌊a/b⌋ × b × y₁ = g
  a × y₁ + b × (x₁ - ⌊a/b⌋ × y₁) = g
  
  Comparing with ax + by = g:
    x = y₁
    y = x₁ - ⌊a/b⌋ × y₁
```

---

## 2.3 Step-by-Step Trace: extGCD(35, 15)

```
  ═══ Forward Pass (standard Euclidean) ═══
  
  Call 1: extGCD(35, 15)
    35 = 15 × 2 + 5    → calls extGCD(15, 5)
  
  Call 2: extGCD(15, 5)
    15 = 5 × 3 + 0     → calls extGCD(5, 0)
  
  Call 3: extGCD(5, 0)
    b = 0 → return (5, 1, 0)    // 5 × 1 + 0 × 0 = 5


  ═══ Backward Pass (compute x, y) ═══
  
  Return to Call 2: extGCD(15, 5)
    g = 5, x₁ = 1, y₁ = 0
    x = y₁ = 0
    y = x₁ - ⌊15/5⌋ × y₁ = 1 - 3 × 0 = 1
    Return (5, 0, 1)
    Verify: 15 × 0 + 5 × 1 = 5 ✓
  
  Return to Call 1: extGCD(35, 15)
    g = 5, x₁ = 0, y₁ = 1
    x = y₁ = 1
    y = x₁ - ⌊35/15⌋ × y₁ = 0 - 2 × 1 = -2
    Return (5, 1, -2)
    Verify: 35 × 1 + 15 × (-2) = 35 - 30 = 5 ✓
  
  RESULT: GCD = 5, x = 1, y = -2
```

### Trace Table

```
  Step │  a   │  b   │  q  │  r  │  x₁ │  y₁ │  x  │  y  │ Verify
  ─────┼──────┼──────┼─────┼─────┼──────┼──────┼─────┼─────┼────────
  Base │   5  │   0  │  —  │  —  │   1  │   0  │  1  │  0  │ 5=5×1
  Back │  15  │   5  │  3  │  0  │   0  │   1  │  0  │  1  │ 5=15×0+5×1
  Back │  35  │  15  │  2  │  5  │   1  │  -2  │  1  │ -2  │ 5=35×1+15×(-2)
```

---

## 2.4 Larger Trace: extGCD(240, 46)

```
  Forward pass:
  240 = 46 × 5 + 10    
   46 = 10 × 4 +  6    
   10 =  6 × 1 +  4    
    6 =  4 × 1 +  2    
    4 =  2 × 2 +  0    → GCD = 2

  Backward pass:
  Step │  a   │  b  │  q  │ x₁  │  y₁ │   x  │   y  │ Check
  ─────┼──────┼─────┼─────┼─────┼──────┼──────┼──────┼──────────
  Base │   2  │  0  │  —  │   1 │    0 │    1 │    0 │ 2 = 2×1
    1  │   4  │  2  │  2  │   0 │    1 │    0 │    1 │ 2 = 4×0 + 2×1
    2  │   6  │  4  │  1  │   1 │   -1 │    1 │   -1 │ 2 = 6×1 + 4×(-1)
    3  │  10  │  6  │  1  │  -1 │    2 │   -1 │    2 │ 2 = 10×(-1) + 6×2
    4  │  46  │ 10  │  4  │   2 │   -9 │    2 │   -9 │ 2 = 46×2 + 10×(-9)
    5  │ 240  │ 46  │  5  │  -9 │   47 │   -9 │   47 │ 2 = 240×(-9) + 46×47

  Final: 240 × (-9) + 46 × 47 = -2160 + 2162 = 2 ✓
```

---

## 2.5 Iterative Version

```
FUNCTION extGCDIterative(a, b):
    old_r, r = a, b
    old_s, s = 1, 0
    old_t, t = 0, 1
    
    WHILE r ≠ 0:
        q = old_r / r
        old_r, r = r, old_r - q × r
        old_s, s = s, old_s - q × s
        old_t, t = t, old_t - q × t
    
    // old_r = GCD, old_s = x, old_t = y
    RETURN (old_r, old_s, old_t)
```

### Trace Table for extGCDIterative(252, 198)

```
  Step │  q  │ old_r │  r   │ old_s │  s  │ old_t │  t
  ─────┼─────┼───────┼──────┼───────┼─────┼───────┼─────
  Init │  —  │  252  │ 198  │   1   │  0  │   0   │  1
    1  │  1  │  198  │  54  │   0   │  1  │   1   │ -1
    2  │  3  │   54  │  36  │   1   │ -3  │  -1   │  4
    3  │  1  │   36  │  18  │  -3   │  4  │   4   │ -5
    4  │  2  │   18  │   0  │   4   │-11  │  -5   │ 14

  Result: GCD = 18, x = 4, y = -5
  Verify: 252 × 4 + 198 × (-5) = 1008 - 990 = 18 ✓
```

---

## 2.6 Finding Modular Inverse

```
  ┌──────────────────────────────────────────────────────────┐
  │  MODULAR INVERSE:                                        │
  │                                                          │
  │  a⁻¹ mod m  exists  ⟺  GCD(a, m) = 1                  │
  │                                                          │
  │  If ax + my = 1  (from Extended GCD)                     │
  │  Then ax ≡ 1 (mod m)                                     │
  │  So x = a⁻¹ mod m                                       │
  └──────────────────────────────────────────────────────────┘
```

### Example: Find 7⁻¹ mod 11

```
  extGCD(7, 11):
  
  7 = 11 × 0 + 7     → extGCD(11, 7)
  11 = 7 × 1 + 4     → extGCD(7, 4)
  7 = 4 × 1 + 3      → extGCD(4, 3)
  4 = 3 × 1 + 1      → extGCD(3, 1)
  3 = 1 × 3 + 0      → GCD = 1

  Back-substitute to get: 7 × (-3) + 11 × 2 = 1
  
  So 7 × (-3) ≡ 1 (mod 11)
  -3 mod 11 = 8
  
  7⁻¹ mod 11 = 8
  Verify: 7 × 8 = 56 = 5 × 11 + 1 ≡ 1 (mod 11) ✓
```

---

## 2.7 Linear Diophantine Equations

$$ax + by = c \text{ has integer solutions } \iff \gcd(a, b) \mid c$$

```
  If GCD(a,b) = g and g | c:
  1. Find x₀, y₀ such that ax₀ + by₀ = g  (using Extended GCD)
  2. Scale: x' = x₀ × (c/g), y' = y₀ × (c/g)
  3. General solution: x = x' + (b/g)t, y = y' - (a/g)t for any integer t

  Example: Solve 6x + 10y = 14
  GCD(6, 10) = 2, and 2 | 14  ✓ (solution exists)
  
  ExtGCD(6, 10): 6 × 2 + 10 × (-1) = 2
  Scale by 14/2 = 7: x₀ = 14, y₀ = -7
  Check: 6(14) + 10(-7) = 84 - 70 = 14 ✓
  
  General: x = 14 + 5t, y = -7 - 3t
  t=0: (14, -7)    t=-1: (9, -4)    t=-2: (4, -1)    t=-3: (-1, 2)
```

---

## 2.8 Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Extended GCD (recursive) | O(log min(a,b)) | O(log min(a,b)) stack |
| Extended GCD (iterative) | O(log min(a,b)) | O(1) |
| Modular inverse | O(log m) | O(1) |
| Diophantine equation | O(log min(a,b)) | O(1) |

---

## Summary Table

| Concept | Formula/Result |
|---------|---------------|
| Bézout's identity | ax + by = GCD(a, b) |
| Back-substitution | x = y₁, y = x₁ - ⌊a/b⌋ × y₁ |
| Modular inverse | From ax + my = 1, inverse = x mod m |
| Diophantine solvability | ax + by = c solvable iff GCD(a,b) \| c |
| General solution | x = x₀ + (b/g)t, y = y₀ - (a/g)t |

---

## Quick Revision Questions

1. **State Bézout's identity** and give an example.
2. **Trace extGCD(56, 15)** — find x, y such that 56x + 15y = GCD.
3. **Find 13⁻¹ mod 47** using the Extended Euclidean Algorithm.
4. **Does 12x + 18y = 7 have integer solutions?** Why or why not?
5. **Solve 15x + 28y = 1** — find all integer solutions.
6. **What is the back-substitution formula** and why does it work?

---

[← Previous: Euclidean Algorithm](01-euclidean-algorithm.md) | [Back to README](../README.md) | [Next: LCM Computation →](03-lcm-computation.md)
