# Chapter 3.3: Highest Common Factor (HCF) and Lowest Common Multiple (LCM)

[← Previous: Prime Factorization](02-prime-factorization.md) | [Back to Contents](../README.md) | [Next: Divisibility Rules →](04-divisibility-rules.md)

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Find HCF using listing, factorization, and division methods
- Find LCM using listing, factorization, and division methods
- Understand the relationship between HCF and LCM
- Apply HCF and LCM to solve real-world problems

---

## 1. What is HCF?

### Definition
The **Highest Common Factor (HCF)** is the largest number that divides two or more numbers exactly.

```
Also known as:
• GCD (Greatest Common Divisor)
• GCF (Greatest Common Factor)
```

### Understanding Through Example
```
Find HCF of 12 and 18:

Factors of 12: {1, 2, 3, 4, 6, 12}
Factors of 18: {1, 2, 3, 6, 9, 18}

Common factors: {1, 2, 3, 6}

Highest common factor: 6

HCF(12, 18) = 6
```

### Visual Representation
```
Factors of 12:    1   2   3   4   6   12
                  |   |   |       |
                  ↓   ↓   ↓       ↓
Common:           1   2   3       6      ← HCF = 6
                  ↑   ↑   ↑       ↑
                  |   |   |       |
Factors of 18:    1   2   3   6   9   18
```

---

## 2. What is LCM?

### Definition
The **Lowest Common Multiple (LCM)** is the smallest positive number that is a multiple of two or more numbers.

### Understanding Through Example
```
Find LCM of 4 and 6:

Multiples of 4: 4, 8, 12, 16, 20, 24, 28, 32, 36, ...
Multiples of 6: 6, 12, 18, 24, 30, 36, ...

Common multiples: 12, 24, 36, ...

Lowest common multiple: 12

LCM(4, 6) = 12
```

### Visual: Number Line Representation
```
Multiples of 4:
|-------|-------|-------|-------|-------|-------|-------|-------|-------|
0       4       8      12      16      20      24      28      32      36

Multiples of 6:
|----------|----------|----------|----------|----------|----------|
0          6         12         18         24         30         36

Common multiples:        12              24              36
                         ↑
                   First common = LCM
```

---

## 3. Method 1: Listing Method

### For HCF
```
Step 1: List all factors of each number
Step 2: Find common factors
Step 3: Select the highest one

Example: HCF(24, 36)

Factors of 24: 1, 2, 3, 4, 6, 8, 12, 24
Factors of 36: 1, 2, 3, 4, 6, 9, 12, 18, 36

Common factors: 1, 2, 3, 4, 6, 12

HCF = 12
```

### For LCM
```
Step 1: List multiples of each number
Step 2: Find the first common multiple

Example: LCM(8, 12)

Multiples of 8:  8, 16, 24, 32, 40, 48, ...
Multiples of 12: 12, 24, 36, 48, ...

First common multiple: 24

LCM = 24
```

### When to Use This Method
```
✓ Good for small numbers
✓ Easy to understand conceptually
✗ Tedious for large numbers
✗ Risk of missing factors or multiples
```

---

## 4. Method 2: Prime Factorization Method

### For HCF
```
Step 1: Find prime factorization of each number
Step 2: Identify COMMON prime factors
Step 3: Take the LOWEST power of each common factor
Step 4: Multiply them together

HCF = Product of common primes with LOWEST powers
```

### Example: HCF(48, 72)
```
48 = 2⁴ × 3
72 = 2³ × 3²

Common primes: 2 and 3

For 2: min(4, 3) = 3 → 2³
For 3: min(1, 2) = 1 → 3¹

HCF = 2³ × 3 = 8 × 3 = 24
```

### For LCM
```
Step 1: Find prime factorization of each number
Step 2: Identify ALL prime factors from both numbers
Step 3: Take the HIGHEST power of each factor
Step 4: Multiply them together

LCM = Product of ALL primes with HIGHEST powers
```

### Example: LCM(48, 72)
```
48 = 2⁴ × 3
72 = 2³ × 3²

All primes: 2 and 3

For 2: max(4, 3) = 4 → 2⁴
For 3: max(1, 2) = 2 → 3²

LCM = 2⁴ × 3² = 16 × 9 = 144
```

### Visual: Comparing HCF and LCM Methods
```
              48 = 2⁴ × 3¹
              72 = 2³ × 3²
              
For HCF:     Take LOWER powers
              2^min(4,3) × 3^min(1,2)
              = 2³ × 3¹ = 24

For LCM:     Take HIGHER powers
              2^max(4,3) × 3^max(1,2)
              = 2⁴ × 3² = 144
```

---

## 5. Method 3: Division Method

### For HCF (Euclidean Algorithm)
```
Step 1: Divide larger number by smaller
Step 2: Replace larger with smaller, smaller with remainder
Step 3: Repeat until remainder is 0
Step 4: The last divisor is the HCF

Example: HCF(48, 72)

72 = 48 × 1 + 24    (72 ÷ 48 = 1 remainder 24)
48 = 24 × 2 + 0     (48 ÷ 24 = 2 remainder 0)

Remainder is 0, so HCF = 24
```

### Visual: Euclidean Algorithm
```
HCF(252, 105)

Step 1: 252 = 105 × 2 + 42
        ↓
Step 2: 105 = 42 × 2 + 21
        ↓
Step 3: 42 = 21 × 2 + 0
        
        Remainder = 0
        HCF = 21

Verify: 252 ÷ 21 = 12 ✓
        105 ÷ 21 = 5 ✓
```

### For LCM (Common Division Method)
```
Step 1: Write numbers side by side
Step 2: Divide by smallest prime that divides at least one number
Step 3: Write quotients below (copy number if not divisible)
Step 4: Repeat until all quotients are 1
Step 5: Multiply all divisors

Example: LCM(12, 18, 24)

 2 | 12  18  24
   |-----|-----|-----
 2 |  6   9  12
   |-----|-----|-----
 2 |  3   9   6
   |-----|-----|-----
 3 |  3   9   3
   |-----|-----|-----
 3 |  1   3   1
   |-----|-----|-----
   |  1   1   1

LCM = 2 × 2 × 2 × 3 × 3 = 72
```

---

## 6. HCF and LCM for Three or More Numbers

### HCF of Multiple Numbers
```
Method: Find HCF of first two, then HCF with third, and so on.

HCF(24, 36, 48)

Step 1: HCF(24, 36)
        24 = 2³ × 3
        36 = 2² × 3²
        HCF = 2² × 3 = 12

Step 2: HCF(12, 48)
        12 = 2² × 3
        48 = 2⁴ × 3
        HCF = 2² × 3 = 12

HCF(24, 36, 48) = 12
```

### LCM of Multiple Numbers
```
LCM(6, 8, 12)

Prime factorizations:
6 = 2 × 3
8 = 2³
12 = 2² × 3

Take highest power of each prime:
2 → max(1, 3, 2) = 3 → 2³
3 → max(1, 0, 1) = 1 → 3¹

LCM = 2³ × 3 = 24
```

---

## 7. The Golden Relationship

### HCF × LCM = Product of Numbers
```
For any two numbers a and b:

HCF(a, b) × LCM(a, b) = a × b

Example: a = 12, b = 18
HCF(12, 18) = 6
LCM(12, 18) = 36

6 × 36 = 216
12 × 18 = 216 ✓
```

### Using This Relationship
```
If HCF and one number known, find LCM:
LCM = (a × b) / HCF

Example: HCF(a, 15) = 5, LCM(a, 15) = ?
If a = 20:
LCM = (20 × 15) / 5 = 300 / 5 = 60
```

### Important Note
```
This relationship ONLY works for TWO numbers!

For three or more numbers:
HCF × LCM ≠ Product of numbers (usually)
```

---

## 8. Co-prime Numbers

### Definition
Two numbers are **co-prime** (or relatively prime) if their HCF is 1.

```
Co-prime examples:
• HCF(8, 15) = 1 → 8 and 15 are co-prime
• HCF(14, 25) = 1 → 14 and 25 are co-prime
• HCF(9, 16) = 1 → 9 and 16 are co-prime

Not co-prime:
• HCF(8, 12) = 4 → Not co-prime
• HCF(15, 25) = 5 → Not co-prime
```

### Properties of Co-prime Numbers
```
1. If a and b are co-prime:
   LCM(a, b) = a × b

2. Consecutive numbers are always co-prime:
   HCF(n, n+1) = 1 for any n

3. Any prime p is co-prime with any number 
   not divisible by p.
```

### Examples
```
8 and 15 are co-prime:
LCM(8, 15) = 8 × 15 = 120

Verify: 
Multiples of 8: 8, 16, 24, ..., 120
Multiples of 15: 15, 30, ..., 120 ✓
```

---

## 9. Applications of HCF and LCM

### HCF Applications

**Problem 1: Cutting Ribbons**
```
Three ribbons of lengths 18m, 24m, and 30m need to be 
cut into equal pieces of maximum length with no waste.
Find the length of each piece.

Solution:
Find HCF(18, 24, 30)

18 = 2 × 3²
24 = 2³ × 3
30 = 2 × 3 × 5

Common factors with lowest powers: 2¹ × 3¹ = 6

Each piece = 6 meters

Pieces from each ribbon: 18/6=3, 24/6=4, 30/6=5
```

**Problem 2: Tiling a Floor**
```
A room is 12m × 15m. What's the largest square tile 
that can be used without cutting?

Solution:
Find HCF(12, 15) = 3

Largest square tile = 3m × 3m
Number of tiles = (12/3) × (15/3) = 4 × 5 = 20 tiles
```

### LCM Applications

**Problem 3: Traffic Lights**
```
Two traffic lights blink every 6 seconds and 8 seconds 
respectively. If they blink together now, when will 
they next blink together?

Solution:
Find LCM(6, 8)

6 = 2 × 3
8 = 2³

LCM = 2³ × 3 = 24

They'll blink together after 24 seconds.
```

**Problem 4: Gears**
```
Two gears have 12 and 18 teeth respectively. 
After how many rotations of the smaller gear 
will they return to starting position?

Solution:
Find LCM(12, 18) = 36

Total teeth engagement: 36
Rotations of 12-tooth gear: 36/12 = 3
Rotations of 18-tooth gear: 36/18 = 2
```

---

## 10. Solved Examples

### Example 1: HCF by Prime Factorization
```
Find HCF(120, 180, 300)

120 = 2³ × 3 × 5
180 = 2² × 3² × 5
300 = 2² × 3 × 5²

Common primes: 2, 3, 5

HCF = 2^min(3,2,2) × 3^min(1,2,1) × 5^min(1,1,2)
    = 2² × 3¹ × 5¹
    = 4 × 3 × 5
    = 60
```

### Example 2: LCM by Prime Factorization
```
Find LCM(15, 20, 24)

15 = 3 × 5
20 = 2² × 5
24 = 2³ × 3

All primes: 2, 3, 5

LCM = 2^max(0,2,3) × 3^max(1,0,1) × 5^max(1,1,0)
    = 2³ × 3¹ × 5¹
    = 8 × 3 × 5
    = 120
```

### Example 3: Using the Relationship
```
If HCF(a, b) = 12 and LCM(a, b) = 144, and a = 48, find b.

Using: HCF × LCM = a × b
12 × 144 = 48 × b
1728 = 48b
b = 36

Verify: HCF(48, 36) = 12 ✓
        LCM(48, 36) = 144 ✓
```

### Example 4: Word Problem
```
The HCF of two numbers is 8 and their LCM is 120.
If one number is 24, find the other.

Solution:
a × b = HCF × LCM
24 × b = 8 × 120
24b = 960
b = 40

The other number is 40.
```

### Example 5: Finding Both HCF and LCM
```
Find HCF and LCM of 126 and 90

Prime factorizations:
126 = 2 × 3² × 7
90 = 2 × 3² × 5

HCF = 2 × 3² = 18 (common primes, lowest powers)
LCM = 2 × 3² × 5 × 7 = 630 (all primes, highest powers)

Check: 18 × 630 = 11340
       126 × 90 = 11340 ✓
```

### Example 6: Euclidean Algorithm
```
Find HCF(456, 168) using division method

456 = 168 × 2 + 120
168 = 120 × 1 + 48
120 = 48 × 2 + 24
48 = 24 × 2 + 0

HCF = 24

Verify: 456/24 = 19, 168/24 = 7 ✓
```

---

## 11. Mental Math Tips 🧠

### Quick HCF for Numbers with Obvious Common Factor
```
HCF(24, 36) → Both divisible by 12 → HCF likely 12
Check: 24 = 2 × 12, 36 = 3 × 12, HCF(2,3) = 1
So HCF = 12 ✓

HCF(35, 49) → Both divisible by 7 → HCF = 7
(35 = 5 × 7, 49 = 7 × 7)
```

### Quick LCM When One Number is Multiple of Other
```
If b divides a evenly:
LCM(a, b) = a

Examples:
LCM(24, 6) = 24 (since 6 divides 24)
LCM(100, 25) = 100 (since 25 divides 100)
```

### Quick LCM for Co-prime Numbers
```
If HCF(a, b) = 1:
LCM(a, b) = a × b

Examples:
LCM(7, 9) = 63 (7 and 9 are co-prime)
LCM(8, 15) = 120 (8 and 15 are co-prime)
```

---

## 📊 Summary Table

### Method Comparison

| Method | Best For | Speed | Accuracy |
|--------|----------|-------|----------|
| Listing | Small numbers | Slow | Easy to miss |
| Prime Factorization | All sizes | Medium | High |
| Division (Euclidean) | Large HCF | Fast | High |
| Common Division | Multiple LCM | Medium | High |

### Key Formulas

| Formula | Description |
|---------|-------------|
| HCF × LCM = a × b | Only for 2 numbers |
| If HCF = 1, LCM = a × b | Co-prime numbers |
| If a\|b, then HCF = a | a divides b |
| If a\|b, then LCM = b | a divides b |

### HCF and LCM at a Glance

```
╔════════════════════╦════════════════════╗
║        HCF         ║        LCM         ║
╠════════════════════╬════════════════════╣
║ Highest Common     ║ Lowest Common      ║
║ Factor             ║ Multiple           ║
╠════════════════════╬════════════════════╣
║ Divides both       ║ Both divide it     ║
╠════════════════════╬════════════════════╣
║ LOWER powers       ║ HIGHER powers      ║
╠════════════════════╬════════════════════╣
║ ≤ smaller number   ║ ≥ larger number    ║
╠════════════════════╬════════════════════╣
║ Cutting/Dividing   ║ Cycles/Timing      ║
║ problems           ║ problems           ║
╚════════════════════╩════════════════════╝
```

---

## ❓ Quick Revision Questions

1. **Find**: HCF(84, 126) using prime factorization.

2. **Find**: LCM(16, 24, 40) using prime factorization.

3. **If** HCF(a, b) = 15 and LCM(a, b) = 180, find a × b.

4. **Problem**: Two bells ring at intervals of 12 and 15 minutes. If they ring together at 9:00 AM, when will they next ring together?

5. **Find**: The largest number that divides 246 and 1030 leaving remainders 6 and 10 respectively.

6. **True or False**: If HCF(a, b) = a, then a divides b.

<details>
<summary>Click to see answers</summary>

1. 84 = 2² × 3 × 7
   126 = 2 × 3² × 7
   **HCF = 2 × 3 × 7 = 42**

2. 16 = 2⁴
   24 = 2³ × 3
   40 = 2³ × 5
   **LCM = 2⁴ × 3 × 5 = 240**

3. a × b = HCF × LCM
   a × b = 15 × 180 = **2700**

4. LCM(12, 15) = 60 minutes
   They'll ring together **60 minutes later at 10:00 AM**

5. The number divides (246-6) = 240 and (1030-10) = 1020
   Find HCF(240, 1020):
   240 = 2⁴ × 3 × 5
   1020 = 2² × 3 × 5 × 17
   HCF = 2² × 3 × 5 = **60**

6. **True**
   If HCF(a, b) = a, then a is a factor of both a and b.
   Since a divides b, the statement is true.

</details>

---

## 🔗 Navigation

[← Previous: Prime Factorization](02-prime-factorization.md) | [Back to Contents](../README.md) | [Next: Divisibility Rules →](04-divisibility-rules.md)
