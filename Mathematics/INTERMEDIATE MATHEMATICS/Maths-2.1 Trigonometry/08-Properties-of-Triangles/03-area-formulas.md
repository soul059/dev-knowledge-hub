# Chapter 8.3: Area Formulas for Triangles

## Overview

There are many ways to calculate the area of a triangle, depending on what information is given. This chapter explores the most important area formulas and their applications.

---

## 📐 Basic Area Formula

$$\boxed{\Delta = \frac{1}{2} \times \text{base} \times \text{height}}$$

```
              C
             /|\
            / | \
           /  |h \ 
          /   |   \
         /    |    \
        A─────D─────B
              base
    
    Area = (1/2) × AB × CD = (1/2) × base × height
```

---

## 📐 Trigonometric Area Formulas

### Using Two Sides and Included Angle

$$\boxed{\Delta = \frac{1}{2}bc\sin A = \frac{1}{2}ca\sin B = \frac{1}{2}ab\sin C}$$

### Proof

```
    From the basic formula:
    
              C
             /|
            / |
         b /  |h
          /   |
         /A   |
        A─────D
    
    Height h = b sin A
    
    Area = (1/2) × base × height
         = (1/2) × c × (b sin A)
         = (1/2) bc sin A  ∎
```

### Example: Find the area of triangle ABC where b = 8, c = 10, and A = 60°.

```
    Δ = (1/2) bc sin A
    Δ = (1/2)(8)(10) sin 60°
    Δ = 40 × (√3/2)
    Δ = 20√3 ≈ 34.64 square units
```

---

## 📐 Heron's Formula (Given Three Sides)

$$\boxed{\Delta = \sqrt{s(s-a)(s-b)(s-c)}}$$

where **s** is the semi-perimeter:

$$s = \frac{a + b + c}{2}$$

### Derivation from Cosine Rule

```
    Starting with Δ = (1/2)bc sin A
    
    We need sin A in terms of sides.
    
    From Cosine Rule: cos A = (b² + c² - a²)/(2bc)
    
    sin²A = 1 - cos²A
    
    sin²A = 1 - [(b² + c² - a²)/(2bc)]²
    
    After algebraic manipulation (quite lengthy):
    
    sin A = (2/bc)√[s(s-a)(s-b)(s-c)]
    
    Therefore:
    Δ = (1/2)bc × (2/bc)√[s(s-a)(s-b)(s-c)]
    Δ = √[s(s-a)(s-b)(s-c)]  ∎
```

### Example: Find the area of a triangle with sides 7, 8, 9.

```
    s = (7 + 8 + 9)/2 = 12
    
    Δ = √[s(s-a)(s-b)(s-c)]
    Δ = √[12(12-7)(12-8)(12-9)]
    Δ = √[12 × 5 × 4 × 3]
    Δ = √720
    Δ = √(144 × 5)
    Δ = 12√5 ≈ 26.83 square units
```

---

## 📐 Area Using Circumradius (R)

$$\boxed{\Delta = \frac{abc}{4R}}$$

### Proof

```
    From Sine Rule: a = 2R sin A
    
    So sin A = a/(2R)
    
    Δ = (1/2)bc sin A
    Δ = (1/2)bc × a/(2R)
    Δ = abc/(4R)  ∎
```

### Example: A triangle has sides 5, 6, 7 and circumradius R = 35/(4√6). Verify the area.

```
    Using Heron's formula:
    s = (5 + 6 + 7)/2 = 9
    Δ = √[9 × 4 × 3 × 2] = √216 = 6√6
    
    Using circumradius formula:
    Δ = abc/(4R) = (5 × 6 × 7)/(4 × 35/(4√6))
    Δ = 210 × (4√6)/(4 × 35)
    Δ = 210√6/35 = 6√6 ✓
```

---

## 📐 Area Using Inradius (r)

$$\boxed{\Delta = rs}$$

where r is the inradius and s is the semi-perimeter.

### Proof

```
              C
             /\
            /  \
           /    \
          / I    \      I = incenter
         /   r    \     r = inradius
        /____●_____\
       A───────────B
    
    The triangle can be divided into three smaller triangles:
    Triangle ABI, Triangle BCI, Triangle CAI
    
    Area of ABI = (1/2) × c × r
    Area of BCI = (1/2) × a × r
    Area of CAI = (1/2) × b × r
    
    Total area = (1/2)r(a + b + c) = r × s  ∎
```

---

## 📐 Area Using All Three Altitudes

If h_a, h_b, h_c are the altitudes to sides a, b, c respectively:

$$\boxed{\Delta = \frac{1}{2}ah_a = \frac{1}{2}bh_b = \frac{1}{2}ch_c}$$

From these, we get:

$$\frac{1}{h_a} + \frac{1}{h_b} + \frac{1}{h_c} = \frac{a + b + c}{2\Delta} = \frac{s}{\Delta} = \frac{1}{r}$$

---

## 📐 Area Using Medians

If m_a, m_b, m_c are the medians from vertices A, B, C:

$$\boxed{\Delta = \frac{4}{3}\sqrt{s_m(s_m-m_a)(s_m-m_b)(s_m-m_c)}}$$

where $s_m = \frac{m_a + m_b + m_c}{2}$

```
    ┌─────────────────────────────────────────────────────────────┐
    │                                                             │
    │   Note: This is like Heron's formula for medians,          │
    │         but with a factor of 4/3                           │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

---

## 📐 Area in Coordinate Geometry

### Given Three Vertices

If A(x₁, y₁), B(x₂, y₂), C(x₃, y₃) are vertices:

$$\boxed{\Delta = \frac{1}{2}|x_1(y_2 - y_3) + x_2(y_3 - y_1) + x_3(y_1 - y_2)|}$$

Or using the determinant form:

$$\Delta = \frac{1}{2}\begin{vmatrix} x_1 & y_1 & 1 \\ x_2 & y_2 & 1 \\ x_3 & y_3 & 1 \end{vmatrix}$$

### Example: Find the area of triangle with vertices A(1, 2), B(4, 6), C(7, 2).

```
    Δ = (1/2)|x₁(y₂ - y₃) + x₂(y₃ - y₁) + x₃(y₁ - y₂)|
    Δ = (1/2)|1(6 - 2) + 4(2 - 2) + 7(2 - 6)|
    Δ = (1/2)|4 + 0 - 28|
    Δ = (1/2)|-24|
    Δ = 12 square units
```

---

## 📐 Special Triangle Areas

### Equilateral Triangle (side a)

$$\boxed{\Delta = \frac{\sqrt{3}}{4}a^2}$$

```
    Proof: Using Δ = (1/2)a² sin 60°
           Δ = (1/2)a² × (√3/2) = (√3/4)a²
```

### Isosceles Triangle (equal sides b, base a)

$$\Delta = \frac{a}{4}\sqrt{4b^2 - a^2}$$

### Right Triangle (legs a, b)

$$\Delta = \frac{1}{2}ab$$

---

## 📝 Worked Examples

### Example 1: Area from Three Angles and One Side

**Find the area of triangle ABC where A = 45°, B = 60°, C = 75°, and c = 10.**

**Solution:**

```
    Using sine rule to find a and b, then area:
    
    c/sin C = a/sin A
    a = c × sin A/sin C = 10 × sin 45°/sin 75°
    
    sin 75° = (√6 + √2)/4 ≈ 0.966
    
    a = 10 × 0.707/0.966 ≈ 7.32
    
    Similarly: b = 10 × sin 60°/sin 75°
    b = 10 × 0.866/0.966 ≈ 8.97
    
    Area = (1/2)ab sin C
         = (1/2)(7.32)(8.97) sin 75°
         = (1/2)(7.32)(8.97)(0.966)
         ≈ 31.7 square units
    
    Alternative (more elegant):
    Using Δ = (c² sin A sin B)/(2 sin C)
    Δ = (100 × sin 45° × sin 60°)/(2 sin 75°)
    Δ = (100 × 0.707 × 0.866)/(2 × 0.966)
    ≈ 31.7 square units
```

---

### Example 2: Finding Missing Side from Area

**The area of triangle ABC is 30 square units. If a = 10 and B = 60°, find c.**

**Solution:**

```
    Δ = (1/2)ac sin B
    30 = (1/2)(10)(c) sin 60°
    30 = 5c × (√3/2)
    30 = (5√3/2)c
    c = 60/(5√3)
    c = 12/√3
    c = 4√3 ≈ 6.93
```

---

### Example 3: Maximum Area Problem

**Two sides of a triangle are 8 and 10. What is the maximum possible area?**

**Solution:**

```
    Area = (1/2)(8)(10) sin θ = 40 sin θ
    
    where θ is the included angle.
    
    Maximum sin θ = 1 when θ = 90°
    
    Maximum area = 40 × 1 = 40 square units
```

This occurs when the triangle is a right triangle with the two given sides as legs.

---

### Example 4: Using Heron's Formula for Complex Problem

**The sides of a triangle are in arithmetic progression with common difference 2. If the area is 6 square units, find the sides.**

**Solution:**

```
    Let sides be a-2, a, a+2
    
    Semi-perimeter: s = (a-2 + a + a+2)/2 = 3a/2
    
    s - (a-2) = 3a/2 - a + 2 = a/2 + 2
    s - a = 3a/2 - a = a/2
    s - (a+2) = 3a/2 - a - 2 = a/2 - 2
    
    By Heron's formula:
    36 = (3a/2)(a/2 + 2)(a/2)(a/2 - 2)
    
    36 = (3a/2)(a/2)[(a/2 + 2)(a/2 - 2)]
    36 = (3a²/4)(a²/4 - 4)
    144 = 3a²(a²/4 - 4)
    48 = a²(a²/4 - 4)
    48 = a⁴/4 - 4a²
    192 = a⁴ - 16a²
    a⁴ - 16a² - 192 = 0
    
    Let x = a²:
    x² - 16x - 192 = 0
    (x - 24)(x + 8) = 0
    x = 24 (since x must be positive)
    
    a² = 24, so a = 2√6
    
    Wait, let's verify: sides would be 2√6 - 2, 2√6, 2√6 + 2
    This gives non-integer sides. Let me recalculate.
    
    Actually, checking with integer sides:
    If sides are 3, 5, 7: s = 7.5
    Δ = √(7.5 × 4.5 × 2.5 × 0.5) = √(42.1875) ≈ 6.49 ≠ 6
    
    If sides are 4, 6, 8: s = 9
    Δ = √(9 × 5 × 3 × 1) = √135 ≈ 11.6 ≠ 6
    
    For exact answer with area = 6:
    The sides are approximately 2.68, 4.68, 6.68
    (or using the exact solution above)
```

---

## 📋 Summary Table

### Area Formulas

| Given Information | Formula | Notes |
|-------------------|---------|-------|
| Base and height | Δ = (1/2)bh | Basic formula |
| Two sides + included angle | Δ = (1/2)bc sin A | Most useful |
| Three sides (SSS) | Δ = √[s(s-a)(s-b)(s-c)] | Heron's formula |
| Sides and circumradius | Δ = abc/(4R) | Uses R |
| Semi-perimeter and inradius | Δ = rs | Uses r |
| Three vertices | Δ = (1/2)\|x₁(y₂-y₃) + x₂(y₃-y₁) + x₃(y₁-y₂)\| | Coordinate geometry |
| Equilateral (side a) | Δ = (√3/4)a² | Special case |

### Relationships Between Area Formulas

```
    ┌─────────────────────────────────────────────────────────────┐
    │                                                             │
    │   From Δ = (1/2)bc sin A and Δ = rs:                       │
    │                                                             │
    │   r = Δ/s = (bc sin A)/(a + b + c)                         │
    │                                                             │
    │   From Δ = abc/(4R):                                        │
    │                                                             │
    │   R = abc/(4Δ)                                              │
    │                                                             │
    │   Combining: r × R = abc × Δ/(4Δs) = abc/(4s)              │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

---

## ❓ Quick Revision Questions

1. **State Heron's formula for the area of a triangle.**

2. **Find the area of a triangle with sides 5, 12, 13.**

3. **Two sides of a triangle are 6 and 8, and the included angle is 30°. Find the area.**

4. **If the area of a triangle is 24 cm² and its sides are 6, 8, 10, find the inradius.**

5. **Express the area of an equilateral triangle with side a in terms of a.**

6. **The circumradius of a triangle is 5 and the sides are 6, 8, 10. Find the area.**

<details>
<summary>Click to see answers</summary>

1. **Heron's formula:**
   $$\Delta = \sqrt{s(s-a)(s-b)(s-c)}$$
   where s = (a+b+c)/2

2. **Area with sides 5, 12, 13:**
   
   s = (5 + 12 + 13)/2 = 15
   
   Δ = √[15(15-5)(15-12)(15-13)]
   Δ = √[15 × 10 × 3 × 2]
   Δ = √900 = **30 square units**
   
   (Note: This is a 5-12-13 right triangle, so Δ = (1/2)(5)(12) = 30 ✓)

3. **Area with sides 6, 8 and included angle 30°:**
   
   Δ = (1/2)(6)(8) sin 30°
   Δ = (1/2)(6)(8)(1/2)
   Δ = **12 square units**

4. **Finding inradius:**
   
   s = (6 + 8 + 10)/2 = 12
   Δ = rs
   24 = r × 12
   **r = 2 cm**

5. **Equilateral triangle area:**
   $$\Delta = \frac{\sqrt{3}}{4}a^2$$

6. **Area using circumradius:**
   
   Δ = abc/(4R)
   Δ = (6 × 8 × 10)/(4 × 5)
   Δ = 480/20 = **24 square units**

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 8.2 Cosine Rule](02-cosine-rule.md) | [Unit 8 Index](README.md) | [8.4 Circumradius and Inradius →](04-circumradius-inradius.md) |

---

[← Back to Main Index](../README.md)
