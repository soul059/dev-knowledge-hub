# Chapter 6.4: Advanced Techniques for Trigonometric Equations

## Overview

This chapter covers advanced methods for solving trigonometric equations that don't fit the simple patterns from previous chapters. We'll explore equations requiring factorization, transformation to a single function, homogeneous equations, and the auxiliary angle method.

---

## 📐 Method 1: Factorization

### When to Use

When the equation can be factored into simpler trigonometric expressions.

### Example 1: sin x cos x = 0

**Solution:**
$$\sin x \cos x = 0$$
$$\sin x = 0 \quad \text{or} \quad \cos x = 0$$

sin x = 0: x = nπ
cos x = 0: x = (2n + 1)π/2

**Combined solution:** x = nπ/2, n ∈ ℤ

### Example 2: sin 2x = sin x

**Solution:**
$$\sin 2x - \sin x = 0$$
$$2\sin x \cos x - \sin x = 0$$
$$\sin x(2\cos x - 1) = 0$$

sin x = 0: x = nπ
cos x = 1/2: x = 2nπ ± π/3

### Example 3: cos 2x = cos x

**Solution:**
$$\cos 2x - \cos x = 0$$
$$2\cos^2 x - 1 - \cos x = 0$$
$$2\cos^2 x - \cos x - 1 = 0$$
$$(2\cos x + 1)(\cos x - 1) = 0$$

cos x = -1/2: x = 2nπ ± 2π/3
cos x = 1: x = 2nπ

---

## 📐 Method 2: Equations of the Form a sin x + b cos x = c

### The Auxiliary Angle Method

For equations of the form **a sin x + b cos x = c**, we use:

$$a\sin x + b\cos x = R\sin(x + \phi)$$

where:
$$R = \sqrt{a^2 + b^2}$$
$$\tan\phi = \frac{b}{a}$$

```
    ┌────────────────────────────────────────────────────────────────┐
    │           AUXILIARY ANGLE METHOD                               │
    ├────────────────────────────────────────────────────────────────┤
    │                                                                │
    │  a sin x + b cos x = c                                        │
    │                                                                │
    │  Step 1: Calculate R = √(a² + b²)                             │
    │                                                                │
    │  Step 2: Find φ where tan φ = b/a                             │
    │                                                                │
    │  Step 3: Rewrite as R sin(x + φ) = c                          │
    │                                                                │
    │  Step 4: sin(x + φ) = c/R                                     │
    │                                                                │
    │  Step 5: Check if |c/R| ≤ 1 (solution exists)                 │
    │                                                                │
    │  Step 6: Solve x + φ = general solution                       │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

### Alternative Form

$$a\sin x + b\cos x = R\cos(x - \psi)$$

where tan ψ = a/b

### Example 4: sin x + cos x = 1

**Solution:**
Here a = 1, b = 1.

$$R = \sqrt{1^2 + 1^2} = \sqrt{2}$$
$$\tan\phi = \frac{1}{1} = 1 \Rightarrow \phi = \frac{\pi}{4}$$

Rewrite:
$$\sqrt{2}\sin\left(x + \frac{\pi}{4}\right) = 1$$
$$\sin\left(x + \frac{\pi}{4}\right) = \frac{1}{\sqrt{2}}$$

Let θ = x + π/4:
$$\sin\theta = \frac{1}{\sqrt{2}}$$
$$\theta = n\pi + (-1)^n \cdot \frac{\pi}{4}$$

Therefore:
$$x + \frac{\pi}{4} = n\pi + (-1)^n \cdot \frac{\pi}{4}$$
$$x = n\pi + (-1)^n \cdot \frac{\pi}{4} - \frac{\pi}{4}$$

**Simplifying:**
- For n even: x = 2kπ + π/4 - π/4 = 2kπ → x = 0, 2π, 4π, ...
- For n odd: x = (2k+1)π - π/4 - π/4 = (2k+1)π - π/2 → x = π/2, 5π/2, ...

**Solutions in [0, 2π]: x = 0, π/2**

### Example 5: √3 sin x + cos x = 2

**Solution:**
Here a = √3, b = 1.

$$R = \sqrt{3 + 1} = 2$$
$$\tan\phi = \frac{1}{\sqrt{3}} \Rightarrow \phi = \frac{\pi}{6}$$

Rewrite:
$$2\sin\left(x + \frac{\pi}{6}\right) = 2$$
$$\sin\left(x + \frac{\pi}{6}\right) = 1$$
$$x + \frac{\pi}{6} = 2n\pi + \frac{\pi}{2}$$
$$x = 2n\pi + \frac{\pi}{2} - \frac{\pi}{6}$$
$$x = 2n\pi + \frac{\pi}{3}$$

### Example 6: sin x + √3 cos x = 1

**Solution:**
Here a = 1, b = √3.

$$R = \sqrt{1 + 3} = 2$$
$$\tan\phi = \sqrt{3} \Rightarrow \phi = \frac{\pi}{3}$$

$$2\sin\left(x + \frac{\pi}{3}\right) = 1$$
$$\sin\left(x + \frac{\pi}{3}\right) = \frac{1}{2}$$
$$x + \frac{\pi}{3} = n\pi + (-1)^n \cdot \frac{\pi}{6}$$
$$x = n\pi + (-1)^n \cdot \frac{\pi}{6} - \frac{\pi}{3}$$

---

## 📐 Method 3: Homogeneous Equations

### Definition

A **homogeneous equation** in sin x and cos x has all terms of the same degree.

### Example: Linear Homogeneous (Degree 1)

a sin x + b cos x = 0

**Solution:** Divide by cos x:
$$a\tan x + b = 0$$
$$\tan x = -\frac{b}{a}$$

### Example 7: 3 sin x - 4 cos x = 0

**Solution:**
Divide by cos x (assuming cos x ≠ 0):
$$3\tan x - 4 = 0$$
$$\tan x = \frac{4}{3}$$
$$x = n\pi + \arctan\left(\frac{4}{3}\right)$$

We need to check if cos x = 0 gives solutions:
If cos x = 0, then sin x = ±1, and 3(±1) - 4(0) = ±3 ≠ 0.
So cos x = 0 is not a solution.

### Example: Quadratic Homogeneous (Degree 2)

a sin²x + b sin x cos x + c cos²x = 0

**Solution:** Divide by cos²x:
$$a\tan^2 x + b\tan x + c = 0$$

### Example 8: 2sin²x - 5sin x cos x + 2cos²x = 0

**Solution:**
Divide by cos²x:
$$2\tan^2 x - 5\tan x + 2 = 0$$
$$(2\tan x - 1)(\tan x - 2) = 0$$
$$\tan x = \frac{1}{2} \quad \text{or} \quad \tan x = 2$$

$$x = n\pi + \arctan\left(\frac{1}{2}\right) \quad \text{or} \quad x = n\pi + \arctan(2)$$

---

## 📐 Method 4: Using Sum-to-Product Formulas

### Example 9: sin 3x + sin x = 0

**Solution:**
Using sum-to-product:
$$2\sin\left(\frac{3x + x}{2}\right)\cos\left(\frac{3x - x}{2}\right) = 0$$
$$2\sin 2x \cos x = 0$$

sin 2x = 0: 2x = nπ → x = nπ/2
cos x = 0: x = (2n + 1)π/2

**Combined:** x = nπ/2

### Example 10: cos 5x - cos x = 0

**Solution:**
Using sum-to-product:
$$-2\sin\left(\frac{5x + x}{2}\right)\sin\left(\frac{5x - x}{2}\right) = 0$$
$$-2\sin 3x \sin 2x = 0$$

sin 3x = 0: 3x = nπ → x = nπ/3
sin 2x = 0: 2x = nπ → x = nπ/2

**Combined solution:** Need to find the union of both sets.

---

## 📐 Method 5: Substitution t = tan(x/2)

### Weierstrass Substitution

Let t = tan(x/2). Then:

$$\sin x = \frac{2t}{1 + t^2}$$
$$\cos x = \frac{1 - t^2}{1 + t^2}$$

This converts all trigonometric equations to polynomial equations in t.

### Example 11: 5cos x + 12sin x = 13

**Solution using auxiliary angle:**
R = √(25 + 144) = 13
$$13\sin(x + \phi) = 13$$
$$\sin(x + \phi) = 1$$

where tan φ = 5/12.

$$x + \phi = 2n\pi + \frac{\pi}{2}$$
$$x = 2n\pi + \frac{\pi}{2} - \arctan\left(\frac{5}{12}\right)$$

---

## 📐 Method 6: Equations with Multiple Angles

### Example 12: sin 2x = cos x

**Solution:**
$$2\sin x \cos x = \cos x$$
$$\cos x(2\sin x - 1) = 0$$

cos x = 0: x = (2n + 1)π/2
sin x = 1/2: x = nπ + (-1)ⁿ(π/6)

### Example 13: cos 3x = 4cos³x - 3cos x (Verification)

This is the triple angle identity, so:
$$4\cos^3 x - 3\cos x = 4\cos^3 x - 3\cos x$$

This is true for all x (identity, not equation).

---

## 📊 Summary of Methods

```
    ┌────────────────────────────────────────────────────────────────┐
    │                    CHOOSING THE RIGHT METHOD                   │
    ├────────────────────────────────────────────────────────────────┤
    │                                                                │
    │  Equation Type                        Method                   │
    │  ─────────────                        ──────                   │
    │                                                                │
    │  Can be factored                      Factor, set each = 0    │
    │                                                                │
    │  a sin x + b cos x = c               Auxiliary angle          │
    │                                                                │
    │  All terms same degree                Divide by cosⁿx         │
    │  (homogeneous)                                                │
    │                                                                │
    │  sin A + sin B or cos A + cos B       Sum-to-product          │
    │                                                                │
    │  Multiple angles like sin 2x          Use double angle        │
    │                                       formulas                │
    │                                                                │
    │  Complex rational expressions         t = tan(x/2)            │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📋 Summary Table

### Method Quick Reference

| Method | When to Use | Key Step |
|--------|-------------|----------|
| Factorization | Terms share common factor | Factor, solve each |
| Auxiliary Angle | a sin x + b cos x = c | R = √(a²+b²), tan φ = b/a |
| Homogeneous | All terms same degree | Divide by cosⁿx |
| Sum-to-Product | Sum/diff of same function | Apply S→P formula |
| Multiple Angle | Contains sin 2x, cos 2x, etc. | Use double/triple formulas |
| Weierstrass | Complex expressions | t = tan(x/2) |

### Auxiliary Angle Formulas

| Form | Transformation |
|------|----------------|
| a sin x + b cos x | R sin(x + φ) where R = √(a²+b²), tan φ = b/a |
| a sin x + b cos x | R cos(x - ψ) where R = √(a²+b²), tan ψ = a/b |
| a cos x + b sin x | R cos(x - φ) where R = √(a²+b²), tan φ = b/a |

---

## ❓ Quick Revision Questions

1. **Solve: sin x cos x = 0 in [0, 2π].**

2. **Solve: sin x + cos x = √2.**

3. **Solve: sin x - √3 cos x = 1 (general solution).**

4. **Solve: 3sin²x + 4sin x cos x - cos²x = 0 in [0, π].**

5. **Solve: sin 5x + sin 3x = 0 (general solution).**

6. **For the equation a sin x + b cos x = c, what is the condition for a solution to exist?**

<details>
<summary>Click to see answers</summary>

1. sin x cos x = 0  
   sin x = 0 or cos x = 0  
   In [0, 2π]:  
   sin x = 0: x = 0, π, 2π  
   cos x = 0: x = π/2, 3π/2  
   **Solutions: x = 0, π/2, π, 3π/2, 2π**

2. sin x + cos x = √2  
   R = √2, tan φ = 1, φ = π/4  
   √2 sin(x + π/4) = √2  
   sin(x + π/4) = 1  
   x + π/4 = 2nπ + π/2  
   **x = 2nπ + π/4**

3. sin x - √3 cos x = 1  
   a = 1, b = -√3, R = 2  
   tan φ = -√3/1 = -√3, so φ = -π/3  
   2 sin(x - π/3) = 1  
   sin(x - π/3) = 1/2  
   x - π/3 = nπ + (-1)ⁿ(π/6)  
   **x = nπ + (-1)ⁿ(π/6) + π/3**

4. 3sin²x + 4sin x cos x - cos²x = 0  
   Divide by cos²x:  
   3tan²x + 4tan x - 1 = 0  
   tan x = (-4 ± √(16 + 12))/6 = (-4 ± √28)/6 = (-4 ± 2√7)/6 = (-2 ± √7)/3  
   
   tan x = (-2 + √7)/3 ≈ 0.215 → x ≈ 0.212 rad  
   tan x = (-2 - √7)/3 ≈ -1.549 → x ≈ 2.142 rad (π - 1.00)  
   
   **In [0, π]: x = arctan((-2+√7)/3) and x = π + arctan((-2-√7)/3)**

5. sin 5x + sin 3x = 0  
   2 sin 4x cos x = 0  
   sin 4x = 0 or cos x = 0  
   
   4x = nπ → x = nπ/4  
   x = (2n+1)π/2  
   
   **General solution: x = nπ/4** (the cos x = 0 solutions are included as n = 2, 6, 10, ...)

6. For a sin x + b cos x = c to have solutions:  
   We rewrite as R sin(x + φ) = c where R = √(a² + b²)  
   Then sin(x + φ) = c/R  
   For sin to have solutions, we need |c/R| ≤ 1  
   So |c| ≤ R  
   **Condition: |c| ≤ √(a² + b²)**  
   Or equivalently: c² ≤ a² + b²

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 6.3 Quadratic Equations](03-quadratic-equations.md) | [Unit 6 Index](README.md) | [Unit 7: Inverse Functions →](../07-Inverse-Trigonometric-Functions/README.md) |

---

[← Back to Main Index](../README.md)
