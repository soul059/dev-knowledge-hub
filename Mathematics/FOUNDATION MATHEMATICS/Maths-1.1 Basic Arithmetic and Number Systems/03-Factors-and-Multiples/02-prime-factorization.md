# Chapter 3.2: Prime Factorization

[← Previous: Prime and Composite Numbers](01-prime-composite-numbers.md) | [Back to Contents](../README.md) | [Next: HCF and LCM →](03-hcf-lcm.md)

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Break down any composite number into prime factors
- Use factor trees and division methods
- Write numbers in standard (index) form
- Apply prime factorization to solve problems

---

## 1. What is Prime Factorization?

### Definition
**Prime factorization** is expressing a number as a product of its prime factors.

```
Every composite number can be written as:
n = p₁^a₁ × p₂^a₂ × p₃^a₃ × ...

where p₁, p₂, p₃, ... are prime numbers
and a₁, a₂, a₃, ... are their powers (exponents)
```

### Examples
```
12 = 2 × 2 × 3 = 2² × 3
30 = 2 × 3 × 5
100 = 2 × 2 × 5 × 5 = 2² × 5²
360 = 2 × 2 × 2 × 3 × 3 × 5 = 2³ × 3² × 5
```

### The Fundamental Theorem of Arithmetic
```
╔═══════════════════════════════════════════════════════╗
║  Every integer greater than 1 has a UNIQUE prime      ║
║  factorization (except for the order of factors).     ║
╚═══════════════════════════════════════════════════════╝

Example: 60 can only be written as 2² × 3 × 5
         No other combination of primes works!
```

---

## 2. Method 1: Factor Tree

### How It Works
```
Start with the number, split it into two factors.
Continue splitting until all factors are prime.

Example: Factor tree for 60

        60
       /  \
      6    10
     / \   / \
    2   3 2   5

60 = 2 × 3 × 2 × 5 = 2² × 3 × 5
```

### Step-by-Step Process
```
Step 1: Write the number at the top
Step 2: Find any two factors (not 1 and the number)
Step 3: Draw branches to these factors
Step 4: If a factor is composite, repeat steps 2-3
Step 5: If a factor is prime, circle it (it's done!)
Step 6: Multiply all circled primes
```

### Example: Factor Tree for 72
```
Method A (starting with 2):

        72
       /  \
      2    36
          /  \
         2    18
             /  \
            2    9
                / \
               3   3

72 = 2 × 2 × 2 × 3 × 3 = 2³ × 3²
```

```
Method B (starting with different factors):

        72
       /  \
      8    9
     /|\   / \
    2 2 2 3   3

72 = 2 × 2 × 2 × 3 × 3 = 2³ × 3²

Same answer! The path doesn't matter.
```

### Example: Factor Tree for 180
```
        180
       /   \
      10    18
     / \   /  \
    2   5 2    9
              / \
             3   3

180 = 2 × 5 × 2 × 3 × 3 = 2² × 3² × 5
```

---

## 3. Method 2: Repeated Division

### How It Works
```
Divide by the smallest prime possible.
Continue until the quotient is 1.

Example: Prime factorization of 60

2 | 60
  -----
2 | 30
  -----
3 | 15
  -----
5 | 5
  -----
  | 1

60 = 2 × 2 × 3 × 5 = 2² × 3 × 5
```

### Step-by-Step Process
```
Step 1: Divide by 2 if even
Step 2: Continue dividing by 2 until odd
Step 3: Try dividing by 3
Step 4: Continue with 5, 7, 11, ... (primes)
Step 5: Stop when quotient = 1
Step 6: Multiply all divisors
```

### Example: Prime Factorization of 840
```
2 | 840
  -----
2 | 420
  -----
2 | 210
  -----
3 | 105
  -----
5 | 35
  -----
7 | 7
  -----
  | 1

840 = 2 × 2 × 2 × 3 × 5 × 7 = 2³ × 3 × 5 × 7
```

### Example: Prime Factorization of 315
```
3 | 315    (315 is odd, so skip 2; try 3)
  -----
3 | 105
  -----
5 | 35
  -----
7 | 7
  -----
  | 1

315 = 3 × 3 × 5 × 7 = 3² × 5 × 7
```

---

## 4. Standard Form (Index Notation)

### Writing in Standard Form
```
Prime factorization in standard form uses exponents:

Expanded Form → Standard Form
2 × 2 × 2 × 2  → 2⁴
2 × 2 × 3 × 3 × 5 → 2² × 3² × 5
2 × 3 × 5 × 7 → 2¹ × 3¹ × 5¹ × 7¹ (or just 2 × 3 × 5 × 7)
```

### Convention
```
Write prime factors in ascending order:
2² × 3² × 5  ✓  (correct)
5 × 3² × 2²  ✗  (unconventional)

Omit exponent 1 (optional):
2 × 3² × 5  or  2¹ × 3² × 5¹
```

### Complete Examples
| Number | Expanded Form | Standard Form |
|--------|---------------|---------------|
| 24 | 2 × 2 × 2 × 3 | 2³ × 3 |
| 36 | 2 × 2 × 3 × 3 | 2² × 3² |
| 100 | 2 × 2 × 5 × 5 | 2² × 5² |
| 144 | 2 × 2 × 2 × 2 × 3 × 3 | 2⁴ × 3² |
| 300 | 2 × 2 × 3 × 5 × 5 | 2² × 3 × 5² |
| 504 | 2 × 2 × 2 × 3 × 3 × 7 | 2³ × 3² × 7 |

---

## 5. Finding Number of Factors

### Formula
If n = p₁^a₁ × p₂^a₂ × p₃^a₃ × ..., then:

```
Number of factors = (a₁ + 1)(a₂ + 1)(a₃ + 1)...
```

### Example: Factors of 60
```
60 = 2² × 3¹ × 5¹

Number of factors = (2+1)(1+1)(1+1)
                  = 3 × 2 × 2
                  = 12

The 12 factors of 60:
1, 2, 3, 4, 5, 6, 10, 12, 15, 20, 30, 60
```

### Example: Factors of 72
```
72 = 2³ × 3²

Number of factors = (3+1)(2+1)
                  = 4 × 3
                  = 12

The 12 factors of 72:
1, 2, 3, 4, 6, 8, 9, 12, 18, 24, 36, 72
```

### Why Does This Work?
```
For 60 = 2² × 3 × 5:

Any factor of 60 has the form: 2^a × 3^b × 5^c
where:
• a can be 0, 1, or 2 (3 choices)
• b can be 0 or 1 (2 choices)
• c can be 0 or 1 (2 choices)

Total combinations = 3 × 2 × 2 = 12

Factor table:
a=0: 1, 3, 5, 15          (2⁰ × 3^b × 5^c)
a=1: 2, 6, 10, 30         (2¹ × 3^b × 5^c)
a=2: 4, 12, 20, 60        (2² × 3^b × 5^c)
```

---

## 6. Perfect Squares and Prime Factorization

### Identifying Perfect Squares
```
A number is a PERFECT SQUARE if and only if
all exponents in its prime factorization are EVEN.

36 = 2² × 3²     All exponents (2, 2) are even → √36 = 6 ✓
144 = 2⁴ × 3²    All exponents (4, 2) are even → √144 = 12 ✓
48 = 2⁴ × 3¹     Exponent 1 is odd → 48 is NOT a perfect square
```

### Finding Square Roots Using Prime Factorization
```
√n = √(p₁^a₁ × p₂^a₂) = p₁^(a₁/2) × p₂^(a₂/2)

Example: √576
576 = 2⁶ × 3²
√576 = 2³ × 3¹ = 8 × 3 = 24

Verify: 24 × 24 = 576 ✓
```

### Making a Number a Perfect Square
```
What's the smallest number to multiply 180 by to make it a perfect square?

180 = 2² × 3² × 5¹

Exponents: 2, 2, 1
           ↓  ↓  ↓
          even even ODD!

We need 5¹ to become 5². Multiply by 5.
180 × 5 = 900 = 2² × 3² × 5² = (2 × 3 × 5)² = 30²
```

---

## 7. Perfect Cubes and Prime Factorization

### Identifying Perfect Cubes
```
A number is a PERFECT CUBE if and only if
all exponents in its prime factorization are MULTIPLES OF 3.

64 = 2⁶           6 is multiple of 3 → ∛64 = 2² = 4 ✓
216 = 2³ × 3³     3, 3 are multiples of 3 → ∛216 = 2 × 3 = 6 ✓
72 = 2³ × 3²      2 is NOT multiple of 3 → 72 is NOT a perfect cube
```

### Finding Cube Roots Using Prime Factorization
```
∛n = ∛(p₁^a₁ × p₂^a₂) = p₁^(a₁/3) × p₂^(a₂/3)

Example: ∛1728
1728 = 2⁶ × 3³
∛1728 = 2² × 3¹ = 4 × 3 = 12

Verify: 12 × 12 × 12 = 1728 ✓
```

---

## 8. Applications of Prime Factorization

### Application 1: Simplifying Fractions
```
Simplify 84/126

Find prime factorizations:
84 = 2² × 3 × 7
126 = 2 × 3² × 7

Cancel common factors:
84/126 = (2² × 3 × 7)/(2 × 3² × 7)
       = (2¹)/(3¹)
       = 2/3
```

### Application 2: Finding LCM and HCF
```
(Covered in detail in next chapter)

HCF uses the LOWER power of common primes.
LCM uses the HIGHER power of all primes.
```

### Application 3: Divisibility Tests
```
Is 1260 divisible by 18?

1260 = 2² × 3² × 5 × 7
18 = 2 × 3²

Check: Does 1260 contain at least 2¹ × 3²?
       1260 has 2² (≥ 2¹) ✓
       1260 has 3² (= 3²) ✓

Yes, 1260 is divisible by 18!
1260 ÷ 18 = 70
```

---

## 9. Solved Examples

### Example 1: Factor Tree for 252
```
        252
       /   \
      4     63
     / \   /  \
    2   2 9    7
         / \
        3   3

252 = 2 × 2 × 3 × 3 × 7 = 2² × 3² × 7
```

### Example 2: Repeated Division for 2940
```
2 | 2940
  -----
2 | 1470
  -----
3 | 735
  -----
5 | 245
  -----
7 | 49
  -----
7 | 7
  -----
  | 1

2940 = 2² × 3 × 5 × 7²
```

### Example 3: Number of Factors of 360
```
360 = ?

2 | 360
  -----
2 | 180
  -----
2 | 90
  -----
3 | 45
  -----
3 | 15
  -----
5 | 5
  -----
  | 1

360 = 2³ × 3² × 5

Number of factors = (3+1)(2+1)(1+1)
                  = 4 × 3 × 2
                  = 24 factors
```

### Example 4: Is 324 a Perfect Square?
```
324 = ?

2 | 324
  -----
2 | 162
  -----
3 | 81
  -----
3 | 27
  -----
3 | 9
  -----
3 | 3
  -----
  | 1

324 = 2² × 3⁴

All exponents (2 and 4) are EVEN.
Yes, 324 is a perfect square!

√324 = 2¹ × 3² = 2 × 9 = 18
```

### Example 5: Smallest Multiplier for Perfect Cube
```
What's the smallest number to multiply 500 by to make it a perfect cube?

500 = 2² × 5³

For perfect cube, all exponents must be multiples of 3.
• 2² needs 2¹ more to become 2³
• 5³ is already a multiple of 3 ✓

Multiply by 2.
500 × 2 = 1000 = 2³ × 5³ = (2 × 5)³ = 10³ ✓
```

### Example 6: Product of Two Numbers
```
If A = 2³ × 3² and B = 2 × 3³ × 5, find A × B.

A × B = (2³ × 3²) × (2 × 3³ × 5)
      = 2³⁺¹ × 3²⁺³ × 5
      = 2⁴ × 3⁵ × 5
      = 16 × 243 × 5
      = 19440
```

---

## 10. Mental Math Tips 🧠

### Quick Division by Small Primes
```
By 2: Even → Yes
By 3: Digit sum divisible by 3 → Yes
By 5: Ends in 0 or 5 → Yes
By 7: No quick rule (must divide)
By 11: Alternating sum test (covered in divisibility)
```

### Recognizing Powers of Primes
```
Powers of 2: 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024
Powers of 3: 3, 9, 27, 81, 243, 729
Powers of 5: 5, 25, 125, 625

These are useful starting points for factorization!
```

### Speed Technique: Factor by Recognition
```
When you see: 36 → Recognize as 6² = 2² × 3²
When you see: 64 → Recognize as 8² = 2⁶ or 4³
When you see: 100 → Recognize as 10² = 2² × 5²
When you see: 144 → Recognize as 12² = 2⁴ × 3²
```

---

## 📊 Summary Table

### Common Prime Factorizations

| Number | Prime Factorization | Number of Factors |
|--------|---------------------|-------------------|
| 12 | 2² × 3 | 6 |
| 18 | 2 × 3² | 6 |
| 24 | 2³ × 3 | 8 |
| 36 | 2² × 3² | 9 |
| 48 | 2⁴ × 3 | 10 |
| 60 | 2² × 3 × 5 | 12 |
| 72 | 2³ × 3² | 12 |
| 100 | 2² × 5² | 9 |
| 120 | 2³ × 3 × 5 | 16 |
| 180 | 2² × 3² × 5 | 18 |
| 360 | 2³ × 3² × 5 | 24 |
| 500 | 2² × 5³ | 12 |

### Special Number Forms

| Type | Condition | Example |
|------|-----------|---------|
| Perfect Square | All exponents even | 36 = 2² × 3² |
| Perfect Cube | All exponents multiple of 3 | 216 = 2³ × 3³ |
| Perfect 4th power | All exponents multiple of 4 | 16 = 2⁴ |

---

## ❓ Quick Revision Questions

1. **Find**: The prime factorization of 168.

2. **Calculate**: How many factors does 120 have?

3. **Determine**: Is 1225 a perfect square? If yes, find its square root.

4. **Find**: The smallest number by which 392 must be multiplied to get a perfect cube.

5. **Simplify**: 270/450 using prime factorization.

6. **If** A = 2⁴ × 3² × 5 and B = 2² × 3³ × 7, find how many factors A has.

<details>
<summary>Click to see answers</summary>

1. 168 = 2³ × 3 × 7
   (168 = 2 × 84 = 2 × 2 × 42 = 2 × 2 × 2 × 21 = 2³ × 3 × 7)

2. 120 = 2³ × 3 × 5
   Factors = (3+1)(1+1)(1+1) = 4 × 2 × 2 = **16 factors**

3. 1225 = 5² × 7² (all exponents even)
   Yes, 1225 is a perfect square.
   √1225 = 5 × 7 = **35**

4. 392 = 2³ × 7²
   For perfect cube: need 7³ (have 7²) and 2³ is fine
   Multiply by 7.
   392 × 7 = 2744 = 2³ × 7³ = **(2 × 7)³ = 14³**

5. 270 = 2 × 3³ × 5
   450 = 2 × 3² × 5²
   270/450 = (2 × 3³ × 5)/(2 × 3² × 5²) = 3/5 = **3/5**

6. A = 2⁴ × 3² × 5
   Factors = (4+1)(2+1)(1+1) = 5 × 3 × 2 = **30 factors**

</details>

---

## 🔗 Navigation

[← Previous: Prime and Composite Numbers](01-prime-composite-numbers.md) | [Back to Contents](../README.md) | [Next: HCF and LCM →](03-hcf-lcm.md)
