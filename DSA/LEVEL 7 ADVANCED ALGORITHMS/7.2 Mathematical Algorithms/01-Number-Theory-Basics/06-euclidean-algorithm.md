# Chapter 6: Euclidean Algorithm

[← Previous: GCD and LCM](05-gcd-and-lcm.md) | [Back to README](../README.md) | [Next: Unit 2 — Primality Check →](../02-Prime-Numbers/01-primality-check-naive.md)

---

## Overview

The Euclidean Algorithm is one of the oldest and most elegant algorithms in mathematics (dating back to ~300 BCE). It computes GCD in O(log n) time using repeated division — far superior to brute-force approaches.

---

## 6.1 Core Idea

```
  ┌──────────────────────────────────────────────────────────┐
  │  KEY INSIGHT:                                            │
  │                                                          │
  │  GCD(a, b) = GCD(b, a mod b)                            │
  │                                                          │
  │  We repeatedly replace the larger number with            │
  │  the remainder of dividing the larger by the smaller,    │
  │  until the remainder becomes 0.                          │
  │  The last non-zero remainder is the GCD.                 │
  └──────────────────────────────────────────────────────────┘
```

### Why Does This Work?

```
  Claim: GCD(a, b) = GCD(b, a mod b)

  Proof:
  Let d = GCD(a, b), so d | a and d | b.

  a = b × q + r   (where r = a mod b)
  ⟹ r = a - b × q
  ⟹ d | r   (since d | a and d | b, d must divide a - bq = r)
  ⟹ d divides both b and r
  ⟹ d ≤ GCD(b, r)

  Conversely, let d' = GCD(b, r), so d' | b and d' | r.
  a = b × q + r ⟹ d' | a
  ⟹ d' divides both a and b
  ⟹ d' ≤ GCD(a, b) = d

  Combined: d ≤ d' and d' ≤ d ⟹ d = d'
  Therefore GCD(a, b) = GCD(b, a mod b)  ∎
```

---

## 6.2 Recursive Implementation

```
FUNCTION gcd(a, b):
    IF b == 0:
        RETURN a
    RETURN gcd(b, a % b)
```

### Step-by-Step Trace: GCD(252, 105)

```
  Call Stack:
  
  gcd(252, 105)
  │  252 % 105 = 42
  └─► gcd(105, 42)
      │  105 % 42 = 21
      └─► gcd(42, 21)
          │  42 % 21 = 0
          └─► gcd(21, 0)
              │  b == 0, return 21
              └─► RETURN 21

  GCD(252, 105) = 21

  Verification: 252 = 21 × 12, 105 = 21 × 5  ✓
```

### Trace Table

```
  Step │  a    │  b    │  a % b  │  Action
  ─────┼───────┼───────┼─────────┼──────────────────
    1  │  252  │  105  │   42    │  gcd(105, 42)
    2  │  105  │   42  │   21    │  gcd(42, 21)
    3  │   42  │   21  │    0    │  gcd(21, 0)
    4  │   21  │    0  │   --    │  return 21
```

---

## 6.3 Iterative Implementation

```
FUNCTION gcd(a, b):
    WHILE b ≠ 0:
        temp = b
        b = a % b
        a = temp
    RETURN a
```

### Trace: GCD(462, 1071)

```
  Step │  a     │  b     │  a % b  
  ─────┼────────┼────────┼────────
    1  │  462   │ 1071   │  462    (swap: a=1071, b=462)
    2  │ 1071   │  462   │  147
    3  │  462   │  147   │   21
    4  │  147   │   21   │    0
    5  │   21   │    0   │   --    → return 21

  GCD(462, 1071) = 21
```

### Visual: How Values Shrink

```
  Values at each step:

  1071 ████████████████████████████████████████████████
   462 ████████████████████
   147 ██████
    21 █
     0 (done)

  Notice: values decrease RAPIDLY (at least halved every 2 steps)
```

---

## 6.4 Complexity Analysis

### Why O(log(min(a, b)))?

```
  Lamé's Theorem:
  The number of steps in the Euclidean algorithm never exceeds
  5 × (number of digits of the smaller number)

  More precisely: ≤ 5 × log₁₀(min(a,b))

  Proof sketch:
  After 2 steps, the remainder is guaranteed to be less than half.
  
  If a ≥ b, then:
    a mod b < a/2  (always true!)

  Why? Two cases:
  Case 1: b ≤ a/2  ⟹  a mod b < b ≤ a/2  ✓
  Case 2: b > a/2  ⟹  a mod b = a - b < a/2  ✓

  So after each step, the value at least halves.
  Number of halvings to reach 0 from n: O(log n)
```

### Worst Case: Fibonacci Numbers!

```
  Fibonacci: 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, ...
  
  GCD(F(n+1), F(n)) requires exactly n steps!
  
  GCD(89, 55):
  89 = 55×1 + 34    → gcd(55, 34)
  55 = 34×1 + 21    → gcd(34, 21)
  34 = 21×1 + 13    → gcd(21, 13)
  21 = 13×1 +  8    → gcd(13, 8)
  13 =  8×1 +  5    → gcd(8, 5)
   8 =  5×1 +  3    → gcd(5, 3)
   5 =  3×1 +  2    → gcd(3, 2)
   3 =  2×1 +  1    → gcd(2, 1)
   2 =  1×2 +  0    → gcd(1, 0) = 1
   
  9 steps for consecutive Fibonacci numbers F(11)=89, F(10)=55
  This is the SLOWEST the algorithm gets for numbers this size.
```

---

## 6.5 Subtraction-Based Version (Original Euclid)

```
  Original Euclidean Algorithm (slower, historical):
  
  FUNCTION gcdSubtract(a, b):
      WHILE a ≠ b:
          IF a > b:
              a = a - b
          ELSE:
              b = b - a
      RETURN a

  Trace: GCD(48, 18)
  48-18=30, 30-18=12, 18-12=6, 12-6=6 → equal! GCD=6
  (4 steps with subtraction vs 3 with modulo)
  
  Time: O(max(a,b)) worst case — much slower!
  The modulo version is a speedup of this.
```

---

## 6.6 GCD with Negative Numbers

```
  Convention: GCD is always non-negative.
  
  GCD(-12, 18) = GCD(12, 18) = 6
  GCD(-12, -18) = GCD(12, 18) = 6
  
  Implementation:
  FUNCTION gcd(a, b):
      a = |a|    // take absolute value
      b = |b|
      WHILE b ≠ 0:
          temp = b
          b = a % b
          a = temp
      RETURN a
```

---

## 6.7 Computing LCM Using GCD

```
  LCM(a, b) = |a × b| / GCD(a, b)

  ⚠️ Overflow Warning:
  a × b can overflow! Use this instead:
  
  LCM(a, b) = (a / GCD(a, b)) × b
  
  Divide first, then multiply to avoid overflow.

  Example: LCM(12, 18)
  GCD(12, 18) = 6
  LCM = (12 / 6) × 18 = 2 × 18 = 36
```

---

## 6.8 Problem-Solving Patterns

### Pattern: GCD of Array

```
  GCD(a₁, a₂, ..., aₙ) = GCD(GCD(...GCD(a₁, a₂), a₃)..., aₙ)
  
  FUNCTION gcdArray(arr):
      result = arr[0]
      FOR i = 1 TO len(arr) - 1:
          result = gcd(result, arr[i])
          IF result == 1: RETURN 1  // early exit optimization
      RETURN result
  
  Trace: GCD(24, 36, 48, 60)
  gcd(24, 36) = 12
  gcd(12, 48) = 12
  gcd(12, 60) = 12
  Result: 12
```

### Pattern: Make Array Elements Equal

```
  Problem: Minimum operations to make all elements equal
           by subtracting any element from another.
  
  Answer: The final equal value = GCD of all elements!
  
  Array: [12, 18, 24]
  GCD(12, 18, 24) = 6
  All elements can eventually become 6.
```

---

## 6.9 Real-World Applications

| Application | How GCD/Euclidean Algorithm Is Used |
|-------------|-------------------------------------|
| Fraction simplification | Divide numerator and denominator by GCD |
| RSA cryptography | Extended Euclidean for modular inverse |
| Music theory | Beat synchronization via LCM |
| Computer graphics | Pixel aspect ratios (GCD of dimensions) |
| Scheduling | Event coincidence via LCM |

---

## Summary Table

| Concept | Formula/Rule | Complexity |
|---------|-------------|------------|
| Euclidean GCD | GCD(a,b) = GCD(b, a%b) | O(log min(a,b)) |
| Base case | GCD(a, 0) = a | O(1) |
| LCM from GCD | (a/GCD) × b | O(log min(a,b)) |
| GCD of array | Reduce pairwise | O(n log M) |
| Worst case inputs | Consecutive Fibonacci numbers | Maximum steps |
| Subtraction version | GCD(a-b, b) | O(max(a,b)) |

---

## Quick Revision Questions

1. **State the key recurrence** of the Euclidean algorithm and prove it.
2. **Trace GCD(270, 192)** step by step using the modulo version.
3. **Why are Fibonacci numbers** the worst-case input for the Euclidean algorithm?
4. **What is Lamé's theorem?** What bound does it give?
5. **How do you compute LCM safely** without integer overflow?
6. **Given the array [36, 60, 84]**, compute the GCD step by step.

---

[← Previous: GCD and LCM](05-gcd-and-lcm.md) | [Back to README](../README.md) | [Next: Unit 2 — Primality Check →](../02-Prime-Numbers/01-primality-check-naive.md)
