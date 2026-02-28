# Chapter 6.4: The Quadratic Formula

[← Previous: Completing the Square](03-completing-the-square.md) | [Back to Contents](../README.md) | [Next: Nature of Roots →](05-nature-of-roots.md)

---

## 📚 Chapter Overview

The **quadratic formula** is the most powerful and versatile method for solving quadratic equations. It works for ALL quadratic equations and directly gives both solutions in terms of the coefficients a, b, and c.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Memorize and apply the quadratic formula
- Identify a, b, and c from any quadratic equation
- Handle all types of solutions (rational, irrational, complex)
- Simplify solutions correctly
- Check solutions by substitution
- Decide when to use the quadratic formula vs. other methods

---

## 1. The Quadratic Formula

### The Formula

```
┌─────────────────────────────────────────────────────────────────────┐
│              THE QUADRATIC FORMULA                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   For the equation  ax² + bx + c = 0  where a ≠ 0:                │
│                                                                     │
│              ┌─────────────────────────────┐                       │
│              │                             │                       │
│              │      -b ± √(b² - 4ac)      │                       │
│              │  x = ─────────────────      │                       │
│              │            2a               │                       │
│              │                             │                       │
│              └─────────────────────────────┘                       │
│                                                                     │
│   This gives TWO solutions:                                        │
│                                                                     │
│        -b + √(b² - 4ac)         -b - √(b² - 4ac)                  │
│   x₁ = ─────────────────   x₂ = ─────────────────                 │
│              2a                       2a                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Memory Aids

```
┌─────────────────────────────────────────────────────────────────────┐
│              WAYS TO REMEMBER THE FORMULA                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Song (to "Pop Goes the Weasel"):                                │
│   "x equals negative b                                             │
│    plus or minus the square root                                  │
│    of b squared minus four a c                                    │
│    all over two a"                                                │
│                                                                     │
│   Mnemonic structure:                                              │
│   "Negative boy couldn't decide (±)                               │
│    whether to go to the radical party (√)                         │
│    The boy was square (b²)                                        │
│    but lost his 4 awesome cats (4ac)                              │
│    And it was all over at 2 am (2a)"                              │
│                                                                     │
│   Visual pattern:                                                  │
│        -b  ±  √(b² - 4ac)                                         │
│        ↑   ↑   ↑     ↑                                            │
│       neg  ± sqrt   discriminant                                  │
│        ─────────────────────                                       │
│              2a                                                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Identifying a, b, and c

### Standard Form Requirement

```
┌─────────────────────────────────────────────────────────────────────┐
│              IDENTIFYING COEFFICIENTS                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   MUST be in standard form: ax² + bx + c = 0                      │
│                                                                     │
│   Examples:                                                        │
│                                                                     │
│   2x² - 5x + 3 = 0       →  a = 2, b = -5, c = 3                 │
│                                                                     │
│   x² + 7x - 2 = 0        →  a = 1, b = 7, c = -2                 │
│                                                                     │
│   -x² + 4x + 5 = 0       →  a = -1, b = 4, c = 5                 │
│                                                                     │
│   3x² - 12 = 0           →  a = 3, b = 0, c = -12                │
│                                                                     │
│   x² + 6x = 0            →  a = 1, b = 6, c = 0                  │
│                                                                     │
│   ⚠️ Watch the SIGNS! Include the negative if present.            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Converting to Standard Form First

```
Equation: 3x = x² - 4

Rearrange: 0 = x² - 3x - 4  or  x² - 3x - 4 = 0

Now: a = 1, b = -3, c = -4
```

---

## 3. Step-by-Step Application

### The Process

```
┌─────────────────────────────────────────────────────────────────────┐
│              USING THE QUADRATIC FORMULA                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Step 1: Write equation in standard form: ax² + bx + c = 0       │
│                                                                     │
│   Step 2: Identify a, b, and c (with their signs!)                │
│                                                                     │
│   Step 3: Substitute into the formula                             │
│                                                                     │
│   Step 4: Calculate the discriminant: b² - 4ac                    │
│                                                                     │
│   Step 5: Simplify under the square root                          │
│                                                                     │
│   Step 6: Calculate both solutions (+ and -)                      │
│                                                                     │
│   Step 7: Simplify final answers                                  │
│                                                                     │
│   Step 8: Verify by substituting back                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. Types of Solutions

### Based on the Discriminant

```
┌─────────────────────────────────────────────────────────────────────┐
│              THE DISCRIMINANT: D = b² - 4ac                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   The value under the square root determines solution types:      │
│                                                                     │
│   D > 0 (positive):                                               │
│   • Two distinct real solutions                                   │
│   • If D is a perfect square → rational solutions                │
│   • If D is not a perfect square → irrational solutions          │
│                                                                     │
│   D = 0 (zero):                                                   │
│   • One real solution (repeated root)                             │
│   • x = -b/(2a)                                                   │
│                                                                     │
│   D < 0 (negative):                                               │
│   • No real solutions                                             │
│   • Two complex conjugate solutions                               │
│                                                                     │
│   ┌─────────────────────────────────────────┐                     │
│   │   D > 0     D = 0      D < 0            │                     │
│   │    ╱╲       ╱╲         ╱╲               │                     │
│   │   ╱  ╲     ╱  ╲       ╱  ╲              │                     │
│   │  ●    ●    ●         (no crossing)      │                     │
│   │  ─────────────────────────              │                     │
│   │  2 roots   1 root     0 real roots     │                     │
│   └─────────────────────────────────────────┘                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5. Simplifying Solutions

### Reducing Fractions

```
┌─────────────────────────────────────────────────────────────────────┐
│              SIMPLIFYING QUADRATIC FORMULA RESULTS                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   After calculating, ALWAYS check if you can simplify!            │
│                                                                     │
│   Example: x = (6 ± √20) / 4                                      │
│                                                                     │
│   Step 1: Simplify the radical                                    │
│           √20 = √(4 × 5) = 2√5                                    │
│                                                                     │
│   Step 2: x = (6 ± 2√5) / 4                                       │
│                                                                     │
│   Step 3: Factor out common factor from numerator                 │
│           x = 2(3 ± √5) / 4                                       │
│                                                                     │
│   Step 4: Reduce the fraction                                     │
│           x = (3 ± √5) / 2                                        │
│                                                                     │
│   ⚠️ You can only cancel if ALL terms share the factor!          │
│                                                                     │
│   WRONG: (6 + √5)/4 ≠ (3 + √5)/2                                 │
│          (Cannot cancel 2 since √5 doesn't have factor of 2)     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. Special Cases

### When b = 0

```
x² - 9 = 0  (a = 1, b = 0, c = -9)

x = (-0 ± √(0 - 4(1)(-9))) / 2(1)
x = (±√36) / 2
x = ±6/2
x = ±3

(Could have used square root method!)
```

### When c = 0

```
x² + 5x = 0  (a = 1, b = 5, c = 0)

x = (-5 ± √(25 - 0)) / 2
x = (-5 ± 5) / 2

x = 0/2 = 0  or  x = -10/2 = -5

(Could have factored: x(x + 5) = 0)
```

---

## ✏️ Solved Examples

### Example 1: Easy - Integer Solutions

**Problem:** Solve x² - 5x + 6 = 0

**Solution:**
```
Identify: a = 1, b = -5, c = 6

Apply formula:
x = (-(-5) ± √((-5)² - 4(1)(6))) / (2(1))
x = (5 ± √(25 - 24)) / 2
x = (5 ± √1) / 2
x = (5 ± 1) / 2

x = (5 + 1)/2 = 6/2 = 3
x = (5 - 1)/2 = 4/2 = 2

Check: 3² - 5(3) + 6 = 9 - 15 + 6 = 0 ✓
       2² - 5(2) + 6 = 4 - 10 + 6 = 0 ✓

Answer: x = 2 or x = 3
```

### Example 2: Easy - Rational Solutions

**Problem:** Solve 2x² + 7x + 3 = 0

**Solution:**
```
Identify: a = 2, b = 7, c = 3

Apply formula:
x = (-7 ± √(49 - 24)) / 4
x = (-7 ± √25) / 4
x = (-7 ± 5) / 4

x = (-7 + 5)/4 = -2/4 = -1/2
x = (-7 - 5)/4 = -12/4 = -3

Answer: x = -1/2 or x = -3
```

### Example 3: Medium - Irrational Solutions

**Problem:** Solve x² + 4x + 1 = 0

**Solution:**
```
Identify: a = 1, b = 4, c = 1

Apply formula:
x = (-4 ± √(16 - 4)) / 2
x = (-4 ± √12) / 2
x = (-4 ± 2√3) / 2
x = -2 ± √3

x = -2 + √3 ≈ -0.268
x = -2 - √3 ≈ -3.732

Answer: x = -2 + √3 or x = -2 - √3
```

### Example 4: Medium - Simplification Needed

**Problem:** Solve 3x² - 6x - 2 = 0

**Solution:**
```
Identify: a = 3, b = -6, c = -2

Discriminant: b² - 4ac = 36 - 4(3)(-2) = 36 + 24 = 60

Apply formula:
x = (6 ± √60) / 6
x = (6 ± √(4 × 15)) / 6
x = (6 ± 2√15) / 6
x = (3 ± √15) / 3

x = (3 + √15)/3 ≈ 2.291
x = (3 - √15)/3 ≈ -0.291

Answer: x = (3 + √15)/3 or x = (3 - √15)/3
```

### Example 5: Medium - One Solution

**Problem:** Solve x² - 6x + 9 = 0

**Solution:**
```
Identify: a = 1, b = -6, c = 9

Discriminant: b² - 4ac = 36 - 36 = 0

Apply formula:
x = (6 ± √0) / 2
x = 6/2
x = 3

This is a repeated root (double root).

Check: (x - 3)² = x² - 6x + 9 ✓

Answer: x = 3 (repeated root)
```

### Example 6: Hard - No Real Solutions

**Problem:** Solve 2x² + x + 3 = 0

**Solution:**
```
Identify: a = 2, b = 1, c = 3

Discriminant: b² - 4ac = 1 - 24 = -23

Since discriminant < 0:
√(-23) is not a real number

Answer: No real solutions

(Complex solutions: x = (-1 ± i√23)/4)
```

### Example 7: Hard - Equation Not in Standard Form

**Problem:** Solve 3x(x - 2) = x - 5

**Solution:**
```
Expand and rearrange:
3x² - 6x = x - 5
3x² - 6x - x + 5 = 0
3x² - 7x + 5 = 0

Identify: a = 3, b = -7, c = 5

Discriminant: b² - 4ac = 49 - 60 = -11

Since D < 0:
Answer: No real solutions
```

### Example 8: Hard - Larger Numbers

**Problem:** Solve 4x² + 12x + 5 = 0

**Solution:**
```
Identify: a = 4, b = 12, c = 5

Discriminant: b² - 4ac = 144 - 80 = 64

Apply formula:
x = (-12 ± √64) / 8
x = (-12 ± 8) / 8

x = (-12 + 8)/8 = -4/8 = -1/2
x = (-12 - 8)/8 = -20/8 = -5/2

Answer: x = -1/2 or x = -5/2
```

---

## ❓ Practice Problems

### Easy Level

1. Solve: x² - 7x + 10 = 0

2. Solve: x² + 2x - 8 = 0

3. Solve: 2x² - 5x - 3 = 0

### Medium Level

4. Solve: x² + 6x + 4 = 0

5. Solve: 3x² - 2x - 2 = 0

6. Solve: x² - 4x + 4 = 0

### Hard Level

7. Solve: 5x² + 3x + 1 = 0

8. Solve: 2x² - 3x - 7 = 0

9. Solve: (x + 2)(x - 1) = 5

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. a=1, b=-7, c=10
   D = 49 - 40 = 9
   x = (7 ± 3)/2
   **x = 5 or x = 2**

2. a=1, b=2, c=-8
   D = 4 + 32 = 36
   x = (-2 ± 6)/2
   **x = 2 or x = -4**

3. a=2, b=-5, c=-3
   D = 25 + 24 = 49
   x = (5 ± 7)/4
   **x = 3 or x = -1/2**

4. a=1, b=6, c=4
   D = 36 - 16 = 20
   x = (-6 ± 2√5)/2
   **x = -3 + √5 or x = -3 - √5**

5. a=3, b=-2, c=-2
   D = 4 + 24 = 28
   x = (2 ± 2√7)/6 = (1 ± √7)/3
   **x = (1 + √7)/3 or x = (1 - √7)/3**

6. a=1, b=-4, c=4
   D = 16 - 16 = 0
   **x = 2 (repeated root)**

7. a=5, b=3, c=1
   D = 9 - 20 = -11 < 0
   **No real solutions**

8. a=2, b=-3, c=-7
   D = 9 + 56 = 65
   **x = (3 + √65)/4 or x = (3 - √65)/4**

9. x² + x - 2 = 5 → x² + x - 7 = 0
   D = 1 + 28 = 29
   **x = (-1 + √29)/2 or x = (-1 - √29)/2**

</details>

---

## 📋 Summary Table

| Discriminant D = b² - 4ac | Number of Real Solutions | Type |
|---------------------------|-------------------------|------|
| D > 0, perfect square | 2 | Rational |
| D > 0, not perfect square | 2 | Irrational |
| D = 0 | 1 | Repeated root |
| D < 0 | 0 | Complex (not real) |

---

## 🔄 Quick Revision Questions

1. **What is the quadratic formula?**

2. **For 3x² - 5x + 2 = 0, what are a, b, and c?**

3. **What is the discriminant?**

4. **If the discriminant is negative, how many real solutions?**

5. **Simplify: (4 ± 2√3)/2**

6. **When can you simplify a quadratic formula answer?**

<details>
<summary>Quick Answers</summary>

1. x = (-b ± √(b² - 4ac)) / (2a)
2. a = 3, b = -5, c = 2
3. D = b² - 4ac
4. Zero real solutions
5. 2 ± √3
6. When ALL terms in the numerator share a common factor with the denominator

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ The Quadratic Formula:                                        │
│                   -b ± √(b² - 4ac)                                 │
│              x = ─────────────────                                 │
│                        2a                                          │
│                                                                     │
│   ★ ALWAYS works for any quadratic equation                       │
│                                                                     │
│   ★ Must be in standard form: ax² + bx + c = 0                    │
│                                                                     │
│   ★ Watch the signs of a, b, c carefully                         │
│                                                                     │
│   ★ Discriminant D = b² - 4ac determines solution type:          │
│     • D > 0: Two real solutions                                   │
│     • D = 0: One repeated solution                                │
│     • D < 0: No real solutions                                    │
│                                                                     │
│   ★ Always simplify radicals and reduce fractions                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Completing the Square](03-completing-the-square.md) | [Back to Contents](../README.md) | [Next: Nature of Roots →](05-nature-of-roots.md)
