# Chapter 6.5: Nature of Roots

[← Previous: The Quadratic Formula](04-quadratic-formula.md) | [Back to Contents](../README.md) | [Next: Quadratic Word Problems →](06-word-problems.md)

---

## 📚 Chapter Overview

The **nature of roots** (also called the "character of roots") of a quadratic equation can be determined without actually solving the equation. The discriminant provides all the information needed to classify the roots as real or complex, rational or irrational, equal or distinct.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Calculate and interpret the discriminant
- Classify roots without solving the equation
- Understand the relationship between roots and graph
- Apply conditions for specific types of roots
- Solve problems involving parameters and root conditions
- Relate the sum and product of roots to coefficients

---

## 1. The Discriminant

### Definition and Meaning

```
┌─────────────────────────────────────────────────────────────────────┐
│              THE DISCRIMINANT                                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   For the quadratic equation ax² + bx + c = 0:                    │
│                                                                     │
│              D = b² - 4ac                                          │
│                                                                     │
│   The discriminant is the expression under the square root        │
│   in the quadratic formula.                                        │
│                                                                     │
│   Since x = (-b ± √D) / (2a), the value of D determines:         │
│                                                                     │
│   • Whether √D is real (D ≥ 0) or imaginary (D < 0)              │
│   • Whether roots are equal (D = 0) or distinct (D ≠ 0)          │
│   • Whether roots are rational (D is perfect square) or not       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Classification of Roots

### Complete Classification

```
┌─────────────────────────────────────────────────────────────────────┐
│              NATURE OF ROOTS BASED ON DISCRIMINANT                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   D = b² - 4ac                                                     │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐ │
│   │                                                             │ │
│   │  D > 0: TWO DISTINCT REAL ROOTS                            │ │
│   │                                                             │ │
│   │     • If D is a perfect square AND a, b, c are rational:  │ │
│   │       Roots are RATIONAL                                   │ │
│   │                                                             │ │
│   │     • If D is not a perfect square OR a, b, c irrational: │ │
│   │       Roots are IRRATIONAL                                 │ │
│   │                                                             │ │
│   └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐ │
│   │                                                             │ │
│   │  D = 0: ONE REPEATED REAL ROOT (equal roots)               │ │
│   │         x = -b/(2a)                                        │ │
│   │                                                             │ │
│   └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐ │
│   │                                                             │ │
│   │  D < 0: NO REAL ROOTS (two complex conjugate roots)        │ │
│   │         Roots are of form p + qi and p - qi                │ │
│   │                                                             │ │
│   └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Visual Summary

```
         DISCRIMINANT D = b² - 4ac
                    │
        ┌───────────┼───────────┐
        │           │           │
     D > 0        D = 0       D < 0
        │           │           │
    Two real    One real     No real
     roots       root         roots
        │           │           │
        │           │      (Complex)
    ┌───┴───┐       │
    │       │       │
 Perfect  Not      │
 square  perfect   │
    │       │       │
Rational Irrational │
```

---

## 3. Geometric Interpretation

### Parabola and X-axis

```
┌─────────────────────────────────────────────────────────────────────┐
│              GRAPHICAL MEANING                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   D > 0: Parabola crosses x-axis at TWO points                    │
│                                                                     │
│         y │    ╱╲                                                  │
│           │   ╱  ╲                                                 │
│      ─────●──────●─────  x-axis                                   │
│           │ ╲  ╱                                                   │
│           │  ╲╱                                                    │
│                                                                     │
│   D = 0: Parabola TOUCHES x-axis at ONE point (vertex)           │
│                                                                     │
│         y │    ╱╲                                                  │
│           │   ╱  ╲                                                 │
│      ─────┼──●────────  x-axis                                    │
│           │   ╲╱                                                   │
│                                                                     │
│   D < 0: Parabola does NOT touch x-axis                           │
│                                                                     │
│         y │    ╱╲                                                  │
│           │   ╱  ╲                                                 │
│           │  ╱    ╲                                                │
│      ─────┼────────────  x-axis                                   │
│           │                                                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. Sum and Product of Roots

### Vieta's Formulas

```
┌─────────────────────────────────────────────────────────────────────┐
│              VIETA'S FORMULAS                                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   For ax² + bx + c = 0 with roots α and β:                        │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐ │
│   │                                                             │ │
│   │   Sum of roots:     α + β = -b/a                           │ │
│   │                                                             │ │
│   │   Product of roots: αβ = c/a                               │ │
│   │                                                             │ │
│   └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│   Derivation:                                                      │
│   If α and β are roots, then:                                     │
│   ax² + bx + c = a(x - α)(x - β)                                 │
│                = a(x² - (α+β)x + αβ)                              │
│                = ax² - a(α+β)x + aαβ                              │
│                                                                     │
│   Comparing coefficients:                                          │
│   b = -a(α+β)  →  α+β = -b/a                                      │
│   c = aαβ      →  αβ = c/a                                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Example

```
For x² - 5x + 6 = 0:

Sum of roots = -(-5)/1 = 5
Product of roots = 6/1 = 6

Actual roots: x = 2 and x = 3
Check: 2 + 3 = 5 ✓
       2 × 3 = 6 ✓
```

---

## 5. Forming Equations from Roots

### Given Roots, Find Equation

```
┌─────────────────────────────────────────────────────────────────────┐
│              FORMING A QUADRATIC FROM ROOTS                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   If α and β are the roots, the equation is:                      │
│                                                                     │
│   x² - (sum of roots)x + (product of roots) = 0                   │
│                                                                     │
│   x² - (α + β)x + αβ = 0                                          │
│                                                                     │
│   ──────────────────────────────────────────────────────────────  │
│                                                                     │
│   Example: Find equation with roots 3 and -2                      │
│                                                                     │
│   Sum = 3 + (-2) = 1                                              │
│   Product = 3 × (-2) = -6                                         │
│                                                                     │
│   Equation: x² - 1x + (-6) = 0                                    │
│             x² - x - 6 = 0                                         │
│                                                                     │
│   Check: (x - 3)(x + 2) = x² - x - 6 ✓                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. Problems with Parameters

### Finding Parameter Values

```
┌─────────────────────────────────────────────────────────────────────┐
│              CONDITION-BASED PROBLEMS                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Common problem types:                                            │
│                                                                     │
│   "Find k such that the equation has equal roots"                 │
│   → Set D = 0                                                      │
│                                                                     │
│   "Find k such that the equation has real roots"                  │
│   → Set D ≥ 0                                                      │
│                                                                     │
│   "Find k such that the equation has no real roots"               │
│   → Set D < 0                                                      │
│                                                                     │
│   "Find k such that the equation has rational roots"              │
│   → Set D = perfect square                                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Example Problem

```
For what value of k does x² + kx + 9 = 0 have equal roots?

For equal roots: D = 0
b² - 4ac = 0
k² - 4(1)(9) = 0
k² - 36 = 0
k² = 36
k = ±6

Verify with k = 6: x² + 6x + 9 = (x + 3)² = 0 ✓
Verify with k = -6: x² - 6x + 9 = (x - 3)² = 0 ✓
```

---

## 7. Relationship Between Roots

### Symmetric Functions of Roots

```
┌─────────────────────────────────────────────────────────────────────┐
│              EXPRESSIONS INVOLVING ROOTS                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Given α + β = -b/a and αβ = c/a, we can find:                   │
│                                                                     │
│   α² + β² = (α + β)² - 2αβ                                        │
│                                                                     │
│   α² - β² = (α + β)(α - β)                                        │
│             where (α - β)² = (α + β)² - 4αβ                       │
│                                                                     │
│   α³ + β³ = (α + β)³ - 3αβ(α + β)                                 │
│                                                                     │
│   1/α + 1/β = (α + β)/(αβ)                                        │
│                                                                     │
│   α/β + β/α = (α² + β²)/(αβ)                                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Example

```
If α and β are roots of x² - 3x + 1 = 0, find α² + β².

From the equation: α + β = 3, αβ = 1

α² + β² = (α + β)² - 2αβ
        = 3² - 2(1)
        = 9 - 2
        = 7
```

---

## ✏️ Solved Examples

### Example 1: Easy - Determine Nature of Roots

**Problem:** Determine the nature of roots of x² - 4x + 3 = 0

**Solution:**
```
a = 1, b = -4, c = 3

D = b² - 4ac = (-4)² - 4(1)(3) = 16 - 12 = 4

Since D = 4 > 0 and 4 is a perfect square:
Answer: Two distinct, rational, real roots

(Verification: x = 1 and x = 3)
```

### Example 2: Easy - No Real Roots

**Problem:** Show that x² + x + 1 = 0 has no real roots

**Solution:**
```
a = 1, b = 1, c = 1

D = b² - 4ac = 1 - 4 = -3

Since D = -3 < 0:
Answer: No real roots (two complex conjugate roots)
```

### Example 3: Medium - Irrational Roots

**Problem:** Determine the nature of roots of x² + 5x + 3 = 0

**Solution:**
```
a = 1, b = 5, c = 3

D = b² - 4ac = 25 - 12 = 13

Since D = 13 > 0 but 13 is NOT a perfect square:
Answer: Two distinct, irrational, real roots

(Roots would be (-5 ± √13)/2)
```

### Example 4: Medium - Parameter for Equal Roots

**Problem:** Find the value of k for which 2x² + kx + 8 = 0 has equal roots

**Solution:**
```
For equal roots: D = 0

a = 2, b = k, c = 8

D = k² - 4(2)(8) = 0
k² - 64 = 0
k² = 64
k = ±8

Verify k = 8: 2x² + 8x + 8 = 0 → x² + 4x + 4 = 0 → (x+2)² = 0 ✓

Answer: k = 8 or k = -8
```

### Example 5: Medium - Sum and Product of Roots

**Problem:** If α and β are roots of 2x² - 5x + 1 = 0, find:
a) α + β
b) αβ
c) α² + β²

**Solution:**
```
a = 2, b = -5, c = 1

a) α + β = -b/a = -(-5)/2 = 5/2

b) αβ = c/a = 1/2

c) α² + β² = (α + β)² - 2αβ
           = (5/2)² - 2(1/2)
           = 25/4 - 1
           = 21/4

Answers: a) 5/2, b) 1/2, c) 21/4
```

### Example 6: Hard - Form Equation from Root Conditions

**Problem:** Find a quadratic equation whose roots are twice the roots of x² - 3x + 2 = 0

**Solution:**
```
First, find roots of original: x² - 3x + 2 = 0
(x - 1)(x - 2) = 0 → x = 1 or x = 2

New roots: 2(1) = 2 and 2(2) = 4

For new equation:
Sum = 2 + 4 = 6
Product = 2 × 4 = 8

Equation: x² - 6x + 8 = 0

Alternatively, using the relationship:
If α, β are original roots, new roots are 2α, 2β
Original: α + β = 3, αβ = 2
New: 2α + 2β = 6, (2α)(2β) = 4αβ = 8

Answer: x² - 6x + 8 = 0
```

### Example 7: Hard - Parameter for Real Roots

**Problem:** Find the range of values of k for which x² + 2kx + k + 6 = 0 has real roots

**Solution:**
```
For real roots: D ≥ 0

a = 1, b = 2k, c = k + 6

D = (2k)² - 4(1)(k + 6) ≥ 0
4k² - 4k - 24 ≥ 0
k² - k - 6 ≥ 0
(k - 3)(k + 2) ≥ 0

This is satisfied when:
k ≤ -2 or k ≥ 3

Answer: k ∈ (-∞, -2] ∪ [3, ∞)
```

### Example 8: Hard - Expression with Roots

**Problem:** If α and β are roots of x² - 4x + 1 = 0, find 1/α + 1/β

**Solution:**
```
From the equation: α + β = 4, αβ = 1

1/α + 1/β = (β + α)/(αβ)
          = (α + β)/(αβ)
          = 4/1
          = 4

Answer: 4
```

---

## ❓ Practice Problems

### Easy Level

1. Determine the nature of roots: x² - 6x + 9 = 0

2. Determine the nature of roots: x² + 2x + 5 = 0

3. Find the sum and product of roots: 3x² - 7x + 2 = 0

### Medium Level

4. For what value of k does x² - 6x + k = 0 have equal roots?

5. Find the values of k for which 2x² + 3x + k = 0 has real roots

6. If the roots of x² + px + 12 = 0 are 3 and 4, find p.

### Hard Level

7. If α, β are roots of x² - 5x + 2 = 0, find α³ + β³

8. Form the equation whose roots are reciprocals of roots of 2x² - 5x + 3 = 0

9. For what values of m does (m-2)x² + 8x + m + 4 = 0 have equal roots?

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. D = 36 - 36 = 0
   **Equal roots (one repeated root, x = 3)**

2. D = 4 - 20 = -16 < 0
   **No real roots (complex)**

3. Sum = 7/3, Product = 2/3
   **Sum = 7/3, Product = 2/3**

4. D = 0: 36 - 4k = 0
   **k = 9**

5. D ≥ 0: 9 - 8k ≥ 0, k ≤ 9/8
   **k ≤ 9/8**

6. Sum = 3 + 4 = 7 = -p/1, so p = -7
   Check: product = 12 ✓
   **p = -7**

7. α + β = 5, αβ = 2
   α³ + β³ = (α + β)³ - 3αβ(α + β) = 125 - 30 = 95
   **95**

8. Original roots: α = 1, β = 3/2
   New roots: 1/α = 1, 1/β = 2/3
   Sum = 5/3, Product = 2/3
   Equation: x² - (5/3)x + 2/3 = 0
   **3x² - 5x + 2 = 0**

9. For equal roots: D = 0
   64 - 4(m-2)(m+4) = 0
   64 - 4(m² + 2m - 8) = 0
   16 - m² - 2m + 8 = 0
   m² + 2m - 24 = 0
   (m + 6)(m - 4) = 0
   m = -6 or m = 4
   Check: m ≠ 2 (or it's not quadratic)
   **m = -6 or m = 4**

</details>

---

## 📋 Summary Table

| Discriminant D | Nature of Roots | Graph Interpretation |
|----------------|-----------------|---------------------|
| D > 0, perfect square | Two distinct rational | Crosses x-axis at 2 points |
| D > 0, not perfect | Two distinct irrational | Crosses x-axis at 2 points |
| D = 0 | One repeated root | Touches x-axis at 1 point |
| D < 0 | No real roots | Doesn't touch x-axis |

| Relationship | Formula |
|--------------|---------|
| Sum of roots | α + β = -b/a |
| Product of roots | αβ = c/a |

---

## 🔄 Quick Revision Questions

1. **What is the formula for the discriminant?**

2. **If D = 49, what type of roots does the equation have?**

3. **If roots are 2 and 5, what's the equation?**

4. **What condition gives equal roots?**

5. **For ax² + bx + c = 0, what is the sum of roots?**

6. **If α + β = 6 and αβ = 8, find α² + β²**

<details>
<summary>Quick Answers</summary>

1. D = b² - 4ac
2. Two distinct rational real roots (49 is a perfect square)
3. x² - 7x + 10 = 0 (sum = 7, product = 10)
4. D = 0
5. -b/a
6. (α + β)² - 2αβ = 36 - 16 = 20

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Discriminant D = b² - 4ac determines root nature             │
│                                                                     │
│   ★ Classification:                                                │
│     D > 0 → Two distinct real roots                               │
│     D = 0 → One repeated root                                     │
│     D < 0 → No real roots                                         │
│                                                                     │
│   ★ Vieta's formulas (for ax² + bx + c = 0):                     │
│     Sum: α + β = -b/a                                             │
│     Product: αβ = c/a                                             │
│                                                                     │
│   ★ To form equation from roots α, β:                            │
│     x² - (α+β)x + αβ = 0                                          │
│                                                                     │
│   ★ Discriminant connects algebra to geometry                     │
│     (number of x-intercepts of parabola)                          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: The Quadratic Formula](04-quadratic-formula.md) | [Back to Contents](../README.md) | [Next: Quadratic Word Problems →](06-word-problems.md)
