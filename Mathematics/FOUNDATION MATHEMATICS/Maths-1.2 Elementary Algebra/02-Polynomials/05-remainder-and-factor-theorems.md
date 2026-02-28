# Chapter 2.5: Remainder and Factor Theorems

[← Previous: Division of Polynomials](04-division-of-polynomials.md) | [Back to Contents](../README.md) | [Next: Common Factor Method →](../03-Factorization/01-common-factor-method.md)

---

## 📚 Chapter Overview

The **Remainder Theorem** and **Factor Theorem** are powerful tools that connect polynomial division with polynomial evaluation. These theorems simplify finding remainders and factors without performing full division.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Apply the Remainder Theorem to find remainders quickly
- Use the Factor Theorem to identify factors
- Find factors of polynomials
- Solve polynomial equations using these theorems

---

## 1. The Remainder Theorem

### Statement

When a polynomial $P(x)$ is divided by $(x - c)$, the **remainder** equals $P(c)$.

$$P(x) = (x - c) \cdot Q(x) + R$$

where $R = P(c)$

```
┌─────────────────────────────────────────────────────────────────────┐
│                  THE REMAINDER THEOREM                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   When P(x) is divided by (x - c):                                 │
│                                                                     │
│   REMAINDER = P(c)                                                 │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────┐      │
│   │                                                         │      │
│   │   To find the remainder when dividing P(x) by (x - c): │      │
│   │                                                         │      │
│   │   Just substitute x = c into P(x)!                     │      │
│   │                                                         │      │
│   │   No division needed!                                   │      │
│   │                                                         │      │
│   └─────────────────────────────────────────────────────────┘      │
│                                                                     │
│   Example: Find remainder when x³ - 4x + 2 is divided by (x - 2)  │
│                                                                     │
│   P(x) = x³ - 4x + 2                                               │
│   P(2) = (2)³ - 4(2) + 2 = 8 - 8 + 2 = 2                          │
│                                                                     │
│   Remainder = 2                                                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Proof of the Remainder Theorem

```
Proof:
─────
By the Division Algorithm, when P(x) is divided by (x - c):

P(x) = (x - c) · Q(x) + R

where R is a constant (since degree of R < degree of (x - c) = 1)

Substituting x = c:
P(c) = (c - c) · Q(c) + R
P(c) = 0 · Q(c) + R
P(c) = R

Therefore, Remainder = P(c)  ∎
```

### Note on Divisor Form

For divisor $(x + a) = (x - (-a))$, use $c = -a$:

| Divisor | Value of c | Remainder |
|---------|------------|-----------|
| $(x - 3)$ | $c = 3$ | $P(3)$ |
| $(x + 2)$ | $c = -2$ | $P(-2)$ |
| $(x - \frac{1}{2})$ | $c = \frac{1}{2}$ | $P(\frac{1}{2})$ |
| $(x + 1)$ | $c = -1$ | $P(-1)$ |

---

## 2. The Factor Theorem

### Statement

$(x - c)$ is a **factor** of $P(x)$ if and only if $P(c) = 0$.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    THE FACTOR THEOREM                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   (x - c) is a factor of P(x)  ⟺  P(c) = 0                        │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────┐      │
│   │                                                         │      │
│   │   If P(c) = 0, then (x - c) divides P(x) exactly       │      │
│   │   (with no remainder)                                   │      │
│   │                                                         │      │
│   │   c is called a ROOT or ZERO of the polynomial         │      │
│   │                                                         │      │
│   └─────────────────────────────────────────────────────────┘      │
│                                                                     │
│   The Factor Theorem is a special case of the Remainder Theorem:  │
│   When the remainder is 0, the divisor is a factor.               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Logical Connection

```
                    REMAINDER THEOREM
                          │
                          ▼
          P(x) ÷ (x - c) has remainder = P(c)
                          │
                          ▼
         ┌────────────────┴────────────────┐
         │                                 │
   If P(c) ≠ 0                      If P(c) = 0
         │                                 │
         ▼                                 ▼
   (x - c) is NOT                  (x - c) IS a
   a factor                        factor of P(x)
                                          │
                                          ▼
                                  FACTOR THEOREM
```

### Proof of the Factor Theorem

```
Proof:
─────
(⟹) If (x - c) is a factor of P(x):
    Then P(x) = (x - c) · Q(x) for some polynomial Q(x)
    Substituting x = c: P(c) = (c - c) · Q(c) = 0

(⟸) If P(c) = 0:
    By the Remainder Theorem, remainder R = P(c) = 0
    So P(x) = (x - c) · Q(x) + 0 = (x - c) · Q(x)
    Therefore (x - c) is a factor of P(x)  ∎
```

---

## 3. Finding Factors Using the Factor Theorem

### Strategy for Finding Rational Roots

```
┌─────────────────────────────────────────────────────────────────────┐
│            RATIONAL ROOT THEOREM (Finding Candidates)               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   For polynomial: P(x) = aₙxⁿ + ... + a₁x + a₀                    │
│                                                                     │
│   If P(x) has a rational root p/q (in lowest terms):              │
│   • p is a factor of the constant term a₀                         │
│   • q is a factor of the leading coefficient aₙ                   │
│                                                                     │
│   Strategy:                                                         │
│   1. List all factors of a₀: ±1, ±2, ±3, ...                      │
│   2. List all factors of aₙ: ±1, ±2, ...                          │
│   3. Test candidates p/q using the Factor Theorem                 │
│   4. If P(p/q) = 0, then (x - p/q) is a factor                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Example: Finding Factors

**Problem:** Find all factors of $P(x) = x³ - 6x² + 11x - 6$

```
Step 1: List possible rational roots
        a₀ = -6: factors are ±1, ±2, ±3, ±6
        aₙ = 1: factors are ±1
        Candidates: ±1, ±2, ±3, ±6

Step 2: Test candidates using Factor Theorem

        P(1) = 1 - 6 + 11 - 6 = 0  ✓ (x - 1) is a factor!
        
Step 3: Divide to find remaining factor
        (x³ - 6x² + 11x - 6) ÷ (x - 1)
        
        Using synthetic division:
        1 │  1   -6    11   -6
          │       1    -5    6
          └────────────────────
             1   -5     6    0
        
        Q(x) = x² - 5x + 6

Step 4: Factor the quotient
        x² - 5x + 6 = (x - 2)(x - 3)
        
        Verify: P(2) = 8 - 24 + 22 - 6 = 0 ✓
                P(3) = 27 - 54 + 33 - 6 = 0 ✓

Result: x³ - 6x² + 11x - 6 = (x - 1)(x - 2)(x - 3)
```

---

## 4. Applications of the Theorems

### Application 1: Finding Unknown Coefficients

If we know that $(x - c)$ is a factor, we can find unknown coefficients.

```
┌─────────────────────────────────────────────────────────────────────┐
│            FINDING UNKNOWN COEFFICIENTS                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   If (x - c) is a factor of P(x), then P(c) = 0                   │
│                                                                     │
│   Example: Find k if (x - 2) is a factor of x³ - 3x² + kx + 4     │
│                                                                     │
│   Solution:                                                         │
│   P(x) = x³ - 3x² + kx + 4                                        │
│   Since (x - 2) is a factor, P(2) = 0                             │
│                                                                     │
│   P(2) = (2)³ - 3(2)² + k(2) + 4 = 0                              │
│        = 8 - 12 + 2k + 4 = 0                                       │
│        = 2k = 0                                                     │
│        k = 0                                                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Application 2: Solving Polynomial Equations

```
To solve P(x) = 0:

1. Find one root c using the Factor Theorem
2. Divide P(x) by (x - c) to get Q(x)
3. Solve Q(x) = 0 (repeat if necessary)
4. The roots are all values where P(x) = 0
```

### Application 3: Checking Divisibility

To check if polynomial A is divisible by polynomial B:
- Find the remainder when A is divided by B
- If remainder = 0, then A is divisible by B

---

## 5. The Generalized Remainder Theorem

### Dividing by Linear $(ax + b)$

When dividing $P(x)$ by $(ax + b)$, the remainder is $P(-\frac{b}{a})$.

```
Example: Find remainder when P(x) = 2x³ - 3x² + 5x - 7 
         is divided by (2x - 1)

Divisor: 2x - 1 = 2(x - 1/2)
So we need P(1/2):

P(1/2) = 2(1/8) - 3(1/4) + 5(1/2) - 7
       = 1/4 - 3/4 + 5/2 - 7
       = 1/4 - 3/4 + 10/4 - 28/4
       = (1 - 3 + 10 - 28)/4
       = -20/4
       = -5

Remainder = -5
```

---

## ✏️ Solved Examples

### Example 1: Easy - Remainder Theorem

**Problem:** Find the remainder when $P(x) = x³ + 2x² - 5x + 3$ is divided by $(x - 1)$.

**Solution:**
```
By the Remainder Theorem, remainder = P(1)

P(1) = (1)³ + 2(1)² - 5(1) + 3
     = 1 + 2 - 5 + 3
     = 1

Remainder = 1
```

### Example 2: Easy - Factor Theorem Check

**Problem:** Is $(x + 2)$ a factor of $x³ + 8$?

**Solution:**
```
For (x + 2) = (x - (-2)), check if P(-2) = 0

P(x) = x³ + 8
P(-2) = (-2)³ + 8
      = -8 + 8
      = 0  ✓

Since P(-2) = 0, (x + 2) IS a factor of x³ + 8.

Note: x³ + 8 = (x + 2)(x² - 2x + 4)
      This is the sum of cubes formula!
```

### Example 3: Medium - Finding Unknown Coefficient

**Problem:** Find the value of k if $(x - 3)$ is a factor of $x³ - kx² + x + 6$.

**Solution:**
```
P(x) = x³ - kx² + x + 6

Since (x - 3) is a factor, P(3) = 0

P(3) = (3)³ - k(3)² + 3 + 6 = 0
     = 27 - 9k + 3 + 6 = 0
     = 36 - 9k = 0
     = 9k = 36
     k = 4

Answer: k = 4

Verification: P(x) = x³ - 4x² + x + 6
             P(3) = 27 - 36 + 3 + 6 = 0 ✓
```

### Example 4: Medium - Complete Factorization

**Problem:** Factorize completely: $P(x) = x³ - 2x² - 5x + 6$

**Solution:**
```
Step 1: Find candidates for rational roots
        Factors of 6: ±1, ±2, ±3, ±6
        Factors of 1: ±1
        Candidates: ±1, ±2, ±3, ±6

Step 2: Test using Factor Theorem
        P(1) = 1 - 2 - 5 + 6 = 0 ✓
        
        So (x - 1) is a factor!

Step 3: Divide by (x - 1)
        1 │  1   -2   -5    6
          │       1   -1   -6
          └────────────────────
             1   -1   -6    0
        
        Q(x) = x² - x - 6

Step 4: Factor the quadratic
        x² - x - 6 = (x - 3)(x + 2)
        
        Check: P(3) = 27 - 18 - 15 + 6 = 0 ✓
               P(-2) = -8 - 8 + 10 + 6 = 0 ✓

Answer: x³ - 2x² - 5x + 6 = (x - 1)(x - 3)(x + 2)
```

### Example 5: Hard - Two Conditions

**Problem:** Find a and b if $P(x) = x³ + ax² + bx + 6$ has $(x - 1)$ and $(x + 2)$ as factors.

**Solution:**
```
Since (x - 1) is a factor: P(1) = 0
Since (x + 2) is a factor: P(-2) = 0

Equation 1: P(1) = 0
1 + a + b + 6 = 0
a + b = -7  ... (i)

Equation 2: P(-2) = 0
(-2)³ + a(-2)² + b(-2) + 6 = 0
-8 + 4a - 2b + 6 = 0
4a - 2b = 2
2a - b = 1  ... (ii)

Solving equations (i) and (ii):
From (i): b = -7 - a
Substitute in (ii): 2a - (-7 - a) = 1
                    2a + 7 + a = 1
                    3a = -6
                    a = -2

From (i): b = -7 - (-2) = -5

Answer: a = -2, b = -5

Verification: P(x) = x³ - 2x² - 5x + 6
             P(1) = 1 - 2 - 5 + 6 = 0 ✓
             P(-2) = -8 - 8 + 10 + 6 = 0 ✓
```

### Example 6: Hard - Solving Polynomial Equation

**Problem:** Solve $x³ - 6x² + 11x - 6 = 0$

**Solution:**
```
P(x) = x³ - 6x² + 11x - 6

Step 1: Test possible roots (factors of 6)
        P(1) = 1 - 6 + 11 - 6 = 0 ✓
        
        So x = 1 is a root, and (x - 1) is a factor

Step 2: Factor using synthetic division
        1 │  1   -6    11   -6
          │       1    -5    6
          └────────────────────
             1   -5     6    0
        
        P(x) = (x - 1)(x² - 5x + 6)

Step 3: Factor the quadratic
        x² - 5x + 6 = 0
        (x - 2)(x - 3) = 0
        x = 2 or x = 3

Complete factorization: (x - 1)(x - 2)(x - 3) = 0

Answer: x = 1, x = 2, x = 3
```

---

## ❓ Practice Problems

### Easy Level

1. Find the remainder when $(x³ - 3x + 5)$ is divided by $(x - 2)$.

2. Is $(x - 1)$ a factor of $x³ - 1$? Justify using the Factor Theorem.

3. Find the remainder when $(2x² + 3x - 4)$ is divided by $(x + 1)$.

### Medium Level

4. Find k if $(x + 1)$ is a factor of $2x³ + x² + kx + 4$.

5. Factorize completely: $x³ - 7x + 6$

6. If $P(x) = x³ + px + q$ and $(x - 1)$ and $(x - 2)$ are both factors, find p and q.

### Hard Level

7. Solve: $2x³ - 5x² - 14x + 8 = 0$

8. Find the common factor of $x³ + x² - x - 1$ and $x³ - x² - x + 1$.

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. P(2) = 8 - 6 + 5 = 7. **Remainder = 7**

2. P(1) = 1 - 1 = 0. **Yes, (x-1) is a factor** ✓

3. P(-1) = 2(1) + 3(-1) - 4 = 2 - 3 - 4 = -5. **Remainder = -5**

4. P(-1) = 0: 2(-1) + 1 + k(-1) + 4 = 0
   -2 + 1 - k + 4 = 0
   3 - k = 0
   **k = 3**

5. Testing: P(1) = 1 - 7 + 6 = 0
   Divide: x³ - 7x + 6 = (x - 1)(x² + x - 6) = **(x - 1)(x + 3)(x - 2)**

6. P(1) = 0: 1 + p + q = 0 → p + q = -1
   P(2) = 0: 8 + 2p + q = 0 → 2p + q = -8
   Solving: **p = -7, q = 6**

7. Test x = 1/2: P(1/2) = 2(1/8) - 5(1/4) - 14(1/2) + 8 = 1/4 - 5/4 - 7 + 8 = 0
   So (2x - 1) is a factor
   Dividing: 2x³ - 5x² - 14x + 8 = (2x - 1)(x² - 2x - 8) = (2x - 1)(x - 4)(x + 2)
   **x = 1/2, 4, -2**

8. For first polynomial: P(1) = 1 + 1 - 1 - 1 = 0, P(-1) = -1 + 1 + 1 - 1 = 0
   For second: Q(1) = 1 - 1 - 1 + 1 = 0, Q(-1) = -1 - 1 + 1 + 1 = 0
   **Common factors: (x - 1) and (x + 1)**

</details>

---

## 📋 Summary Table

| Theorem | Statement | Application |
|---------|-----------|-------------|
| **Remainder Theorem** | Remainder of P(x)÷(x-c) = P(c) | Quick remainder finding |
| **Factor Theorem** | (x-c) is factor ⟺ P(c) = 0 | Testing for factors |
| **Rational Root Theorem** | Possible roots = ±(factors of a₀)/(factors of aₙ) | Finding candidates |

---

## 🔄 Quick Revision Questions

1. **What is the remainder when $P(x)$ is divided by $(x - 5)$?**

2. **If $P(3) = 0$, what can you conclude?**

3. **For divisor $(x + 4)$, what value do you substitute to find the remainder?**

4. **What are the possible rational roots of $x³ + 2x² - 5x - 6$?**

5. **If $P(x) = x² - 5x + 6$, find $P(2)$ and interpret the result.**

6. **Can a cubic polynomial have exactly two real roots?**

<details>
<summary>Quick Answers</summary>

1. Remainder = $P(5)$
2. $(x - 3)$ is a factor of $P(x)$
3. $x = -4$ (since $x + 4 = x - (-4)$)
4. $±1, ±2, ±3, ±6$ (factors of 6 over factors of 1)
5. $P(2) = 4 - 10 + 6 = 0$, so $(x - 2)$ is a factor
6. No, a cubic has either 1 or 3 real roots (counting multiplicity)

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Remainder Theorem: Remainder of P(x)÷(x-c) equals P(c)        │
│                                                                     │
│   ★ Factor Theorem: (x-c) is a factor if and only if P(c) = 0     │
│                                                                     │
│   ★ To find factors, test values from Rational Root Theorem       │
│                                                                     │
│   ★ Once you find one factor, divide and continue factoring       │
│                                                                     │
│   ★ The roots of P(x) = 0 are the values where P(x) has factors   │
│                                                                     │
│   ★ These theorems save time compared to full polynomial division │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Unit 2 Complete!

Congratulations! You have completed **Unit 2: Polynomials**. You now understand:

- ✅ Types and Classification of Polynomials
- ✅ Addition and Subtraction of Polynomials
- ✅ Multiplication of Polynomials
- ✅ Division of Polynomials
- ✅ Remainder and Factor Theorems

**Next up:** [Unit 3: Factorization](../03-Factorization/01-common-factor-method.md)

---

[← Previous: Division of Polynomials](04-division-of-polynomials.md) | [Back to Contents](../README.md) | [Next: Common Factor Method →](../03-Factorization/01-common-factor-method.md)
