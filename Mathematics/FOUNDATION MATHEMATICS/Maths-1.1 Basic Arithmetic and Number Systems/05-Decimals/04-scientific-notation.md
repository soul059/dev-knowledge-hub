# Chapter 5.4: Scientific Notation

[← Previous: Rounding Decimals](03-rounding-decimals.md) | [Back to Contents](../README.md) | [Next: Understanding Percentages →](../06-Percentages/01-understanding-percentages.md)

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Write numbers in scientific notation
- Convert between standard and scientific notation
- Perform calculations with scientific notation
- Understand when to use scientific notation

---

## 1. What is Scientific Notation?

### Definition
**Scientific notation** expresses numbers as a product of a number between 1 and 10, and a power of 10.

```
╔═══════════════════════════════════════════════════════╗
║                                                       ║
║         a × 10ⁿ                                       ║
║                                                       ║
║  where: 1 ≤ a < 10   and   n is an integer          ║
║                                                       ║
╚═══════════════════════════════════════════════════════╝
```

### Why Use Scientific Notation?
```
Large numbers:
Speed of light = 300,000,000 m/s
               = 3 × 10⁸ m/s  ← Much easier!

Small numbers:
Size of atom = 0.0000000001 m
             = 1 × 10⁻¹⁰ m   ← Much clearer!
```

---

## 2. Structure of Scientific Notation

### The Components
```
        5.67 × 10⁴
         ↑     ↑ ↑
         │     │ └── Exponent (power)
         │     └──── Base (always 10)
         └────────── Coefficient (mantissa)
                     Must be 1 ≤ a < 10
```

### Valid vs. Invalid Forms
```
Valid scientific notation:
3.5 × 10⁵     ✓ (3.5 is between 1 and 10)
7.02 × 10⁻³   ✓ (7.02 is between 1 and 10)
1.0 × 10⁰     ✓ (1.0 is between 1 and 10)

Invalid scientific notation:
35 × 10⁴      ✗ (35 is not between 1 and 10)
0.35 × 10⁶    ✗ (0.35 is less than 1)
```

---

## 3. Converting Large Numbers

### Standard Form to Scientific Notation
```
Step 1: Move decimal point left until only one digit before it
Step 2: Count the number of places moved
Step 3: That count becomes the positive exponent

45,000,000 = ?

4.5000000
↑
Moved 7 places left

45,000,000 = 4.5 × 10⁷
```

### Examples of Large Numbers
```
5,000 = 5 × 10³              (moved 3 left)
72,000 = 7.2 × 10⁴           (moved 4 left)
3,450,000 = 3.45 × 10⁶       (moved 6 left)
123,000,000 = 1.23 × 10⁸     (moved 8 left)
8,900,000,000 = 8.9 × 10⁹    (moved 9 left)
```

### Visual Process
```
Converting 6,700,000:

6,700,000.     (start: decimal after last digit)
↑
6,700,00.0     move 1
6,700,0.00     move 2
6,700.000      move 3
6,70.0000      move 4
6,7.00000      move 5
6.700000       move 6 ✓

6,700,000 = 6.7 × 10⁶
```

---

## 4. Converting Small Numbers

### Standard Form to Scientific Notation
```
Step 1: Move decimal point right until one digit before it
Step 2: Count the number of places moved
Step 3: That count becomes the negative exponent

0.00034 = ?

0.00034
    ↑
Move to: 3.4
Moved 4 places right

0.00034 = 3.4 × 10⁻⁴
```

### Examples of Small Numbers
```
0.005 = 5 × 10⁻³              (moved 3 right)
0.072 = 7.2 × 10⁻²            (moved 2 right)
0.000045 = 4.5 × 10⁻⁵         (moved 5 right)
0.00000123 = 1.23 × 10⁻⁶      (moved 6 right)
0.0000000089 = 8.9 × 10⁻⁹     (moved 9 right)
```

### Visual Process
```
Converting 0.000567:

0.000567       (start)
0.00567        move 1 right
0.0567         move 2 right
0.567          move 3 right
5.67           move 4 right ✓

0.000567 = 5.67 × 10⁻⁴
```

---

## 5. Converting Back to Standard Form

### Positive Exponents
```
Move decimal point RIGHT by the exponent value.

3.45 × 10⁵ = 345,000
             ↑
             Move 5 places right (add zeros if needed)

2.7 × 10³ = 2,700
8.04 × 10⁶ = 8,040,000
```

### Negative Exponents
```
Move decimal point LEFT by the exponent value.

3.45 × 10⁻³ = 0.00345
              ↑
              Move 3 places left (add zeros if needed)

5.6 × 10⁻² = 0.056
9.1 × 10⁻⁵ = 0.000091
```

### Memory Aid
```
Positive exponent → BIG number (move right)
Negative exponent → small number (move left)

Think: "Positive = Power = Plus zeros"
       "Negative = Nano = Near zero"
```

---

## 6. Multiplying in Scientific Notation

### The Rule
```
(a × 10ᵐ) × (b × 10ⁿ) = (a × b) × 10ᵐ⁺ⁿ

Multiply coefficients, ADD exponents!
```

### Example 1
```
(3 × 10⁴) × (2 × 10³)

= (3 × 2) × 10⁴⁺³
= 6 × 10⁷
```

### Example 2
```
(4.5 × 10⁶) × (2 × 10⁻²)

= (4.5 × 2) × 10⁶⁺⁽⁻²⁾
= 9 × 10⁴
```

### Example 3: Adjusting Result
```
(5 × 10³) × (4 × 10²)

= (5 × 4) × 10³⁺²
= 20 × 10⁵   ← Not proper form! (20 > 10)

Adjust: 20 = 2 × 10¹
= 2 × 10¹ × 10⁵
= 2 × 10⁶
```

---

## 7. Dividing in Scientific Notation

### The Rule
```
(a × 10ᵐ) ÷ (b × 10ⁿ) = (a ÷ b) × 10ᵐ⁻ⁿ

Divide coefficients, SUBTRACT exponents!
```

### Example 1
```
(8 × 10⁷) ÷ (2 × 10³)

= (8 ÷ 2) × 10⁷⁻³
= 4 × 10⁴
```

### Example 2
```
(6.4 × 10⁵) ÷ (3.2 × 10⁻²)

= (6.4 ÷ 3.2) × 10⁵⁻⁽⁻²⁾
= 2 × 10⁵⁺²
= 2 × 10⁷
```

### Example 3: Adjusting Result
```
(3 × 10⁵) ÷ (6 × 10²)

= (3 ÷ 6) × 10⁵⁻²
= 0.5 × 10³   ← Not proper form! (0.5 < 1)

Adjust: 0.5 = 5 × 10⁻¹
= 5 × 10⁻¹ × 10³
= 5 × 10²
```

---

## 8. Adding and Subtracting

### The Rule
```
Exponents must be the SAME before adding or subtracting!

Then add/subtract coefficients and keep the exponent.
```

### Example 1: Same Exponent
```
(3.2 × 10⁵) + (4.5 × 10⁵)

= (3.2 + 4.5) × 10⁵
= 7.7 × 10⁵
```

### Example 2: Different Exponents
```
(5.2 × 10⁶) + (3.4 × 10⁵)

Step 1: Make exponents same
3.4 × 10⁵ = 0.34 × 10⁶

Step 2: Add
(5.2 + 0.34) × 10⁶ = 5.54 × 10⁶
```

### Example 3: Subtraction
```
(8.5 × 10⁴) - (2.3 × 10³)

Step 1: Make exponents same
2.3 × 10³ = 0.23 × 10⁴

Step 2: Subtract
(8.5 - 0.23) × 10⁴ = 8.27 × 10⁴
```

---

## 9. Real-World Applications

### Astronomy
```
Distance to Sun: 1.5 × 10⁸ km
Distance to nearest star: 4 × 10¹³ km
Diameter of Milky Way: 9.5 × 10¹⁷ km
```

### Physics
```
Speed of light: 3 × 10⁸ m/s
Electron mass: 9.1 × 10⁻³¹ kg
Planck's constant: 6.63 × 10⁻³⁴ J·s
```

### Chemistry
```
Avogadro's number: 6.02 × 10²³ particles/mol
Size of atom: ~1 × 10⁻¹⁰ m
Size of nucleus: ~1 × 10⁻¹⁵ m
```

### Biology
```
Number of cells in human body: ~3.7 × 10¹³
Diameter of red blood cell: 7 × 10⁻⁶ m
Bacteria size: 1 × 10⁻⁶ m
```

### Computing
```
1 Kilobyte: 10³ bytes
1 Megabyte: 10⁶ bytes
1 Gigabyte: 10⁹ bytes
1 Terabyte: 10¹² bytes
```

---

## 10. Solved Examples

### Example 1: Convert to Scientific Notation
```
Convert 456,000,000 to scientific notation:

4.56 × 10⁸
(Moved decimal 8 places left)
```

### Example 2: Convert from Scientific Notation
```
Convert 2.35 × 10⁻⁵ to standard form:

0.0000235
(Moved decimal 5 places left)
```

### Example 3: Multiplication
```
(2.5 × 10⁴) × (3.0 × 10⁻²)

= (2.5 × 3.0) × 10⁴⁺⁽⁻²⁾
= 7.5 × 10²
= 750
```

### Example 4: Division
```
(9.6 × 10⁸) ÷ (4.0 × 10²)

= (9.6 ÷ 4.0) × 10⁸⁻²
= 2.4 × 10⁶
```

### Example 5: Real-World Problem
```
Light travels at 3 × 10⁸ m/s.
How far does light travel in 5 minutes?

Time = 5 min = 300 s = 3 × 10² s

Distance = Speed × Time
         = (3 × 10⁸) × (3 × 10²)
         = 9 × 10¹⁰ meters
         = 90,000,000,000 meters
         = 90 billion meters!
```

### Example 6: Comparing Sizes
```
How many times larger is a meter than a nanometer?

1 meter = 1 × 10⁰ m
1 nanometer = 1 × 10⁻⁹ m

Ratio = (1 × 10⁰) ÷ (1 × 10⁻⁹)
      = 1 × 10⁰⁻⁽⁻⁹⁾
      = 1 × 10⁹

A meter is 1 billion times larger than a nanometer!
```

---

## 11. Mental Math Tips 🧠

### Quick Conversions
```
Know your zeros:
10³ = 1,000 (3 zeros)
10⁶ = 1,000,000 (6 zeros)
10⁹ = 1,000,000,000 (9 zeros)

10⁻³ = 0.001 (move 3 left)
10⁻⁶ = 0.000001 (move 6 left)
```

### Prefix Reference
```
kilo (k) = 10³
mega (M) = 10⁶
giga (G) = 10⁹
tera (T) = 10¹²

milli (m) = 10⁻³
micro (μ) = 10⁻⁶
nano (n) = 10⁻⁹
pico (p) = 10⁻¹²
```

### Quick Multiplication Check
```
When multiplying: ADD exponents
3 × 10⁵ times 2 × 10³ ≈ 6 × 10⁸

When dividing: SUBTRACT exponents
8 × 10⁷ divided by 2 × 10³ ≈ 4 × 10⁴
```

---

## 📊 Summary Table

### Conversion Reference

| Standard Form | Scientific Notation | Places Moved |
|---------------|---------------------|--------------|
| 5,000 | 5 × 10³ | 3 left |
| 450,000 | 4.5 × 10⁵ | 5 left |
| 0.005 | 5 × 10⁻³ | 3 right |
| 0.00045 | 4.5 × 10⁻⁴ | 4 right |

### Operations Summary

| Operation | Rule | Example |
|-----------|------|---------|
| Multiply | Add exponents | 10³ × 10² = 10⁵ |
| Divide | Subtract exponents | 10⁵ ÷ 10² = 10³ |
| Add/Subtract | Same exponent needed | Match powers first |

### Powers of 10

| Power | Value | Name |
|-------|-------|------|
| 10¹² | 1,000,000,000,000 | Trillion |
| 10⁹ | 1,000,000,000 | Billion |
| 10⁶ | 1,000,000 | Million |
| 10³ | 1,000 | Thousand |
| 10⁰ | 1 | One |
| 10⁻³ | 0.001 | Thousandth |
| 10⁻⁶ | 0.000001 | Millionth |
| 10⁻⁹ | 0.000000001 | Billionth |

---

## ❓ Quick Revision Questions

1. **Convert** 78,500,000 to scientific notation.

2. **Convert** 3.06 × 10⁻⁵ to standard form.

3. **Calculate**: (4 × 10⁵) × (3 × 10⁻²)

4. **Calculate**: (9 × 10⁸) ÷ (3 × 10⁵)

5. **Calculate**: (2.5 × 10⁴) + (3.0 × 10³)

6. **Problem**: A virus has diameter 1.5 × 10⁻⁷ m. Express this in nanometers (1 nm = 10⁻⁹ m).

<details>
<summary>Click to see answers</summary>

1. 78,500,000 = **7.85 × 10⁷**
   (Move decimal 7 places left)

2. 3.06 × 10⁻⁵ = **0.0000306**
   (Move decimal 5 places left)

3. (4 × 10⁵) × (3 × 10⁻²)
   = (4 × 3) × 10⁵⁺⁽⁻²⁾
   = **12 × 10³ = 1.2 × 10⁴**

4. (9 × 10⁸) ÷ (3 × 10⁵)
   = (9 ÷ 3) × 10⁸⁻⁵
   = **3 × 10³**

5. Convert 3.0 × 10³ = 0.3 × 10⁴
   (2.5 + 0.3) × 10⁴ = **2.8 × 10⁴**

6. 1.5 × 10⁻⁷ m = ? nm
   = 1.5 × 10⁻⁷ ÷ 10⁻⁹ nm
   = 1.5 × 10⁻⁷⁺⁹ nm
   = 1.5 × 10² nm
   = **150 nanometers**

</details>

---

## 🔗 Navigation

[← Previous: Rounding Decimals](03-rounding-decimals.md) | [Back to Contents](../README.md) | [Next: Understanding Percentages →](../06-Percentages/01-understanding-percentages.md)
