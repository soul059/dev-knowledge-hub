# Chapter 1.3: Irrational Numbers

[← Previous: Rational Numbers](02-rational-numbers-fractions.md) | [Back to Contents](../README.md) | [Next: Real Number System →](04-real-number-system.md)

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Define irrational numbers and understand their characteristics
- Identify common irrational numbers (√2, π, e)
- Prove that certain numbers are irrational
- Represent irrational numbers on a number line
- Perform operations with irrational numbers

---

## 1. What are Irrational Numbers?

### Definition
An **irrational number** is a real number that **cannot** be expressed as a fraction p/q where p and q are integers and q ≠ 0.

In other words: Numbers that are NOT rational are called irrational.

### Key Characteristics
1. **Cannot be written as a simple fraction**
2. **Decimal expansion is non-terminating and non-repeating**
3. **Fills the "gaps" between rational numbers on the number line**

### Notation
The set of irrational numbers is often denoted as:
- ℚ' (complement of rationals)
- ℝ - ℚ (real minus rational)
- Sometimes 𝕀 or 𝕁

---

## 2. Examples of Irrational Numbers

### Famous Irrational Numbers

| Number | Approximate Value | Description |
|--------|-------------------|-------------|
| √2 | 1.41421356237... | Length of diagonal of unit square |
| √3 | 1.73205080757... | Height of equilateral triangle |
| √5 | 2.23606797750... | Related to golden ratio |
| π (pi) | 3.14159265359... | Ratio of circumference to diameter |
| e | 2.71828182846... | Base of natural logarithm |
| φ (phi) | 1.61803398875... | Golden ratio = (1+√5)/2 |

### Decimal Patterns

**Rational Number (Terminating)**:
```
1/4 = 0.25         ← Ends
```

**Rational Number (Repeating)**:
```
1/3 = 0.333333...  ← Pattern "3" repeats
1/7 = 0.142857142857... ← Pattern "142857" repeats
```

**Irrational Number (Non-terminating, Non-repeating)**:
```
√2 = 1.41421356237309504880168872420969807856967187537694...
     ↑ No pattern ever repeats!

π = 3.14159265358979323846264338327950288419716939937510...
    ↑ Goes on forever with no repeating pattern
```

### Creating Irrational Numbers
You can construct irrational numbers:
```
0.101001000100001000001...
  ↑ Pattern changes (increasing zeros)
  
0.123456789101112131415161718192021...
  ↑ Champernowne constant (all positive integers listed)
```

---

## 3. Proving Numbers are Irrational

### Proof by Contradiction: √2 is Irrational

This is one of the most famous proofs in mathematics!

**Theorem**: √2 is irrational

**Proof**:
```
Assume √2 IS rational (we'll show this leads to contradiction)

Step 1: If √2 is rational, then √2 = p/q where:
        • p and q are integers
        • q ≠ 0
        • p/q is in lowest terms (HCF(p,q) = 1)

Step 2: Square both sides
        2 = p²/q²
        2q² = p²

Step 3: This means p² is even (since 2q² is even)
        If p² is even, then p must be even
        (because odd² = odd, so if p² is even, p is even)

Step 4: Let p = 2k for some integer k
        Substitute: 2q² = (2k)²
                    2q² = 4k²
                    q² = 2k²

Step 5: This means q² is even, so q is even

Step 6: CONTRADICTION!
        Both p and q are even
        This means they share factor 2
        But we assumed HCF(p,q) = 1 ✗

Step 7: Our assumption must be wrong
        Therefore, √2 is IRRATIONAL ∎
```

### General Pattern
The same proof works for √3, √5, √6, √7, √8, √10, etc.

**Key Insight**: √n is irrational if n is NOT a perfect square.

```
Irrational: √2, √3, √5, √6, √7, √8, √10, √11, √12, √13...
Rational:   √1=1, √4=2, √9=3, √16=4, √25=5, √36=6...
            (Perfect squares have rational square roots)
```

---

## 4. Identifying Irrational Numbers

### Quick Test: Is it Irrational?

```
┌─────────────────────────────────────────────────────────┐
│                  Is the number IRRATIONAL?              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  √n where n is NOT a perfect square ──────────► YES     │
│                                                         │
│  π, e, φ (golden ratio) ──────────────────────► YES     │
│                                                         │
│  Non-repeating, non-terminating decimal ─────► YES      │
│                                                         │
│  Sum of rational and irrational ─────────────► YES      │
│     Example: 2 + √3                                     │
│                                                         │
│  Product of non-zero rational and irrational ─► YES     │
│     Example: 5√2                                        │
│                                                         │
│  Can be written as a/b (a,b integers, b≠0) ──► NO       │
│                                                         │
│  Terminating decimal ────────────────────────► NO       │
│                                                         │
│  Repeating decimal ──────────────────────────► NO       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Examples: Classify as Rational or Irrational

| Number | Classification | Reason |
|--------|---------------|--------|
| √16 | Rational | √16 = 4 = 4/1 |
| √17 | Irrational | 17 is not a perfect square |
| 0.75 | Rational | = 3/4 |
| 0.757575... | Rational | Repeating decimal = 75/99 = 25/33 |
| 0.12122122212222... | Irrational | Non-repeating pattern |
| π/2 | Irrational | π times any non-zero rational is irrational |
| √2 + √8 | Irrational | = √2 + 2√2 = 3√2 (still irrational) |
| √2 × √2 | Rational | = 2 |
| 22/7 | Rational | It's a fraction (commonly used as π approximation) |

---

## 5. Representing Irrational Numbers on Number Line

### Method: Geometric Construction

We can locate irrational numbers using the Pythagorean theorem:
```
a² + b² = c²
```

### Constructing √2 on the Number Line

**Step-by-Step Construction**:
```
Step 1: Draw a unit square (side = 1)
        Diagonal² = 1² + 1² = 2
        Diagonal = √2

Step 2: Use compass to transfer this length to number line

        B
        │╲
        │ ╲  √2
      1 │  ╲
        │   ╲
        ├────A
           1
        
Number Line:
    0         1    √2    2
    |─────────|─────|─────|
              ├─────┘
               Use compass
               to mark √2
```

### Constructing √3 on the Number Line
```
Step 1: Create right triangle with legs 1 and √2
        Hypotenuse² = 1² + (√2)² = 1 + 2 = 3
        Hypotenuse = √3

Step 2: Use compass to mark on number line

            C
           /|
      √3  / |
         /  | 1
        /   |
       B────A
         √2

Number Line:
    0         1    √2  √3   2
    |─────────|─────|───|───|
```

### Constructing √5 on the Number Line
```
Right triangle with legs 1 and 2:
Hypotenuse² = 1² + 2² = 5
Hypotenuse = √5

Number Line:
    0    1    2   √5   3
    |────|────|────|───|
```

### General Pattern: The Spiral of Theodorus
```
Starting from a right triangle with legs 1 and 1:

         √5
        ╱│
       ╱ │1
      ╱  │      √4=2
    √6   │     ╱│
     ╲   │    ╱ │1
      ╲  │  √3  │
       ╲ │ ╱│   │
        ╲│╱ │1  │
       √2   │   │
         ╲  │   │
          ╲ │1  │
           ╲│───┴─
            1

This spiral construction locates:
√2, √3, √4(=2), √5, √6, √7, ... on the plane
```

### Approximate Positions on Number Line
```
    0       1   √2  √3    2      √5      √6  √7  √8  3     π
    |───────|────|───|────|───────|───────|───|───|──|─────|───►
            1  1.41 1.73  2     2.24    2.45 2.65 2.83    3.14
```

---

## 6. Operations with Irrational Numbers

### Addition and Subtraction

**Like Terms** (same irrational part): Can be combined
```
3√2 + 5√2 = 8√2
7√3 - 2√3 = 5√3
```

**Unlike Terms**: Cannot be simplified further
```
√2 + √3 ≈ 1.414 + 1.732 ≈ 3.146 (irrational)
2√5 - 3√7 (cannot simplify - different surds)
```

**Rational + Irrational = Irrational**
```
3 + √2 ≈ 4.414... (irrational)
5 - π ≈ 1.858... (irrational)
```

### Multiplication

**Irrational × Irrational**: May be rational or irrational
```
√2 × √2 = 2 (rational!)
√2 × √3 = √6 (irrational)
π × π = π² (irrational)
```

**Rational × Irrational = Irrational** (if rational ≠ 0)
```
5 × √3 = 5√3 (irrational)
1/2 × π = π/2 (irrational)
0 × √2 = 0 (rational - special case)
```

### Division

**Irrational ÷ Irrational**: May be rational or irrational
```
√8 ÷ √2 = √(8/2) = √4 = 2 (rational!)
√6 ÷ √2 = √3 (irrational)
π ÷ π = 1 (rational!)
```

### Summary of Operations

| Operation | Result | Example |
|-----------|--------|---------|
| Rational + Irrational | Irrational | 3 + √2 |
| Irrational + Irrational | Usually Irrational | √2 + √3, but √2 + (-√2) = 0 |
| Rational × Irrational (≠0) | Irrational | 5√3 |
| Irrational × Irrational | May be either | √2 × √2 = 2, √2 × √3 = √6 |
| Irrational ÷ Irrational | May be either | √8 ÷ √2 = 2, √6 ÷ √2 = √3 |

---

## 7. Simplifying Square Roots

### Method: Factor out perfect squares

```
√n = √(a² × b) = a√b
```

### Examples
```
√8 = √(4 × 2) = √4 × √2 = 2√2
√12 = √(4 × 3) = 2√3
√18 = √(9 × 2) = 3√2
√45 = √(9 × 5) = 3√5
√72 = √(36 × 2) = 6√2
√50 = √(25 × 2) = 5√2
```

### Step-by-Step Process
```
Simplify √48:

Step 1: Find the largest perfect square factor of 48
        48 = 16 × 3  (16 is a perfect square)
        
Step 2: Split the square root
        √48 = √(16 × 3) = √16 × √3
        
Step 3: Simplify
        √48 = 4√3

Verification: (4√3)² = 16 × 3 = 48 ✓
```

### Common Simplifications Table

| Original | Simplified | Process |
|----------|------------|---------|
| √8 | 2√2 | √(4×2) |
| √12 | 2√3 | √(4×3) |
| √18 | 3√2 | √(9×2) |
| √20 | 2√5 | √(4×5) |
| √27 | 3√3 | √(9×3) |
| √32 | 4√2 | √(16×2) |
| √45 | 3√5 | √(9×5) |
| √48 | 4√3 | √(16×3) |
| √50 | 5√2 | √(25×2) |
| √75 | 5√3 | √(25×3) |
| √98 | 7√2 | √(49×2) |

---

## 8. Solved Examples

### Example 1: Classify each number as rational or irrational

a) √36
b) √0.04
c) √0.9
d) 3.14
e) 3.14159...
f) √(4/9)

**Solution**:
```
a) √36 = 6 = 6/1 → RATIONAL

b) √0.04 = √(4/100) = 2/10 = 0.2 → RATIONAL

c) √0.9 = √(9/10) = 3/√10 → IRRATIONAL
   (since √10 is irrational)

d) 3.14 = 314/100 = 157/50 → RATIONAL
   (terminating decimal)

e) 3.14159... (if non-repeating, like π) → IRRATIONAL

f) √(4/9) = √4/√9 = 2/3 → RATIONAL
```

### Example 2: Simplify √72 + √32 - √18

**Solution**:
```
Step 1: Simplify each term
        √72 = √(36 × 2) = 6√2
        √32 = √(16 × 2) = 4√2
        √18 = √(9 × 2) = 3√2

Step 2: Combine like terms
        6√2 + 4√2 - 3√2
        = (6 + 4 - 3)√2
        = 7√2

Answer: 7√2
```

### Example 3: Find two irrational numbers between 2 and 3

**Solution**:
```
Method 1: Use square roots
2² = 4, 3² = 9
So √5, √6, √7, √8 are all between 2 and 3

Answer: √5 ≈ 2.236 and √7 ≈ 2.646

Method 2: Add irrational to rational
2 + 0.1√2 ≈ 2.141
2 + 0.5√2 ≈ 2.707

Answer: 2 + 0.1√2 and 2 + 0.5√2
```

### Example 4: Simplify (3√5 + 2√3)(3√5 - 2√3)

**Solution**:
```
This is in the form (a + b)(a - b) = a² - b²

Let a = 3√5, b = 2√3

(3√5 + 2√3)(3√5 - 2√3)
= (3√5)² - (2√3)²
= 9 × 5 - 4 × 3
= 45 - 12
= 33

Answer: 33 (a rational number!)
```

### Example 5: If √5 = 2.236, find the value of √125

**Solution**:
```
√125 = √(25 × 5)
     = √25 × √5
     = 5 × √5
     = 5 × 2.236
     = 11.180

Answer: 11.180
```

### Example 6: Find three irrational numbers between √2 and √3

**Solution**:
```
√2 ≈ 1.414
√3 ≈ 1.732

We need numbers x where 1.414 < x < 1.732
Check: √2 < √2.1 < √2.5 < √2.9 < √3

Answer: √2.1, √2.5, √2.9

Alternative: ∜5 (fourth root of 5)
∜5 = 5^(1/4) ≈ 1.495

Answer: √2.1 ≈ 1.449, √2.5 ≈ 1.581, √2.9 ≈ 1.703
```

---

## 9. Mental Math Tricks 🧠

### Trick 1: Estimating Square Roots
```
For √50:
• 7² = 49 (close!)
• 8² = 64
• √50 ≈ 7.07

For √75:
• 8² = 64
• 9² = 81
• √75 is closer to 9
• √75 ≈ 8.66
```

### Trick 2: Quick Check for Perfect Squares
```
Perfect squares end in: 0, 1, 4, 5, 6, or 9

If a number ends in 2, 3, 7, or 8, it's NOT a perfect square
→ Its square root is IRRATIONAL

Example: √47 must be irrational (47 ends in 7)
```

### Trick 3: Simplifying Products of Surds
```
√a × √b = √(ab)

√2 × √8 = √16 = 4
√3 × √12 = √36 = 6
√5 × √20 = √100 = 10
```

### Trick 4: Recognizing Conjugates
```
When you see (a + √b)(a - √b):
Result = a² - b (always rational!)

(5 + √3)(5 - √3) = 25 - 3 = 22
(√7 + √2)(√7 - √2) = 7 - 2 = 5
```

---

## 10. π (Pi) - A Special Irrational Number

### What is π?
```
π = Circumference / Diameter (of any circle)

π = C/d for ALL circles!

π = 3.14159265358979323846264338327950288...
```

### Properties of π
- **Irrational**: Cannot be expressed as a fraction
- **Transcendental**: Not the solution of any polynomial equation with integer coefficients
- **Universal**: Same value for every circle in the universe
- **Digits**: Over 100 trillion digits have been calculated!

### Common Approximations
| Approximation | Value | Error |
|---------------|-------|-------|
| 3 | 3.000 | 4.5% |
| 22/7 | 3.142857... | 0.04% |
| 355/113 | 3.1415929... | 0.00008% |
| 3.14159 | 3.14159 | 0.00001% |

### Why 22/7 ≠ π
```
22/7 = 3.142857142857142857... (repeating)
π    = 3.14159265358979323846... (non-repeating)

22/7 is a RATIONAL approximation
π is IRRATIONAL
They are NOT equal!
```

---

## 📊 Summary Table

### Classification Guide

| Category | Can be written as p/q? | Decimal Type | Examples |
|----------|------------------------|--------------|----------|
| Rational | Yes | Terminating or Repeating | 1/2, 0.75, 0.333... |
| Irrational | No | Non-terminating, Non-repeating | √2, π, e |

### Common Irrational Numbers

| Symbol | Name | Approximate Value | Used In |
|--------|------|-------------------|---------|
| √2 | Square root of 2 | 1.41421 | Geometry, construction |
| √3 | Square root of 3 | 1.73205 | Equilateral triangles |
| π | Pi | 3.14159 | Circles, spheres |
| e | Euler's number | 2.71828 | Exponential growth |
| φ | Golden ratio | 1.61803 | Art, nature, architecture |

### Operations Summary

| If you have... | The result is... |
|----------------|------------------|
| Rational + Rational | Rational |
| Rational + Irrational | Irrational |
| Irrational + Irrational | May be either |
| Rational × Irrational (≠0) | Irrational |
| Irrational × Irrational | May be either |

---

## ❓ Quick Revision Questions

1. **True or False**: 22/7 is equal to π.

2. **Classify**: Is √0.25 rational or irrational?

3. **Simplify**: √50 + √18 - √8

4. **Find**: Two irrational numbers between 3 and 4.

5. **Calculate**: (√7 + √5)(√7 - √5)

6. **Prove**: Show that √3 is irrational (using contradiction method).

<details>
<summary>Click to see answers</summary>

1. **False** - 22/7 ≈ 3.142857... (repeating), but π = 3.14159... (non-repeating)

2. **Rational** - √0.25 = √(1/4) = 1/2 = 0.5

3. √50 + √18 - √8
   = 5√2 + 3√2 - 2√2
   = **(5 + 3 - 2)√2 = 6√2**

4. **√10 ≈ 3.162** and **√11 ≈ 3.317** (both between 3 and 4)
   Or: **π ≈ 3.14159** and **3 + 0.5√2 ≈ 3.707**

5. Using (a+b)(a-b) = a² - b²:
   (√7 + √5)(√7 - √5) = (√7)² - (√5)² = 7 - 5 = **2**

6. **Proof**:
   Assume √3 = p/q (lowest terms)
   Then 3 = p²/q², so 3q² = p²
   This means p² is divisible by 3, so p is divisible by 3
   Let p = 3k, then 3q² = 9k², so q² = 3k²
   This means q is also divisible by 3
   Contradiction: Both p and q divisible by 3, but we assumed lowest terms
   Therefore, √3 is irrational ∎

</details>

---

## 🔗 Navigation

[← Previous: Rational Numbers](02-rational-numbers-fractions.md) | [Back to Contents](../README.md) | [Next: Real Number System →](04-real-number-system.md)
