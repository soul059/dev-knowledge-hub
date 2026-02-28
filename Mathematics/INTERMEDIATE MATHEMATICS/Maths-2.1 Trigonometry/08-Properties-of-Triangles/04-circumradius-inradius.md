# Chapter 8.4: Circumradius, Inradius, and Related Properties

## Overview

Every triangle has special circles associated with it: the circumcircle (passing through all vertices) and the incircle (tangent to all sides). The radii of these circles reveal deep connections between triangle elements.

---

## 📐 The Circumcircle and Circumradius

### Definition

The **circumcircle** is the unique circle passing through all three vertices of a triangle. Its center is called the **circumcenter** (O), and its radius is the **circumradius** (R).

```
              * * *
          *           *
        *       A       *
       *       / \       *
      *       /   \       *
     *       /  O  \       *
     *      /   ●   \      *
      *    /         \    *
       *  B───────────C  *
        *               *
          *           *
              * * *
    
    O = Circumcenter (equidistant from A, B, C)
    R = OA = OB = OC
```

### Formulas for Circumradius

$$\boxed{R = \frac{a}{2\sin A} = \frac{b}{2\sin B} = \frac{c}{2\sin C}}$$

$$\boxed{R = \frac{abc}{4\Delta}}$$

where Δ is the area of the triangle.

### Derivation

From the Sine Rule: $\frac{a}{\sin A} = 2R$

Therefore: $R = \frac{a}{2\sin A}$

For the second formula:
```
    Δ = (1/2)bc sin A
    sin A = 2Δ/(bc)
    
    R = a/(2 sin A) = a/(2 × 2Δ/(bc)) = abc/(4Δ)
```

---

## 📐 Location of Circumcenter

The circumcenter is the intersection of perpendicular bisectors of the sides.

### Position Relative to Triangle

| Triangle Type | Circumcenter Location |
|---------------|----------------------|
| Acute triangle | Inside the triangle |
| Right triangle | On the hypotenuse (midpoint) |
| Obtuse triangle | Outside the triangle |

```
    Acute:              Right:              Obtuse:
    
       A                   A                    A
      /|\                  |\                  / \
     / | \                 | \                /   \
    /  ●  \               ●───\              /     \
   /   O   \              O   B             /       \
  B─────────C              \               B─────────C
                            C                   ●O
```

---

## 📐 The Incircle and Inradius

### Definition

The **incircle** is the largest circle that fits inside the triangle, tangent to all three sides. Its center is the **incenter** (I), and its radius is the **inradius** (r).

```
              A
             /\
            /  \
           /    \
          /  ●I  \      I = Incenter
         /   /\   \     r = perpendicular distance
        /   / r\   \        from I to each side
       /   /    \   \
      B───●──────●───C
```

### Formulas for Inradius

$$\boxed{r = \frac{\Delta}{s}}$$

where Δ is area and s is semi-perimeter.

$$\boxed{r = (s-a)\tan\frac{A}{2} = (s-b)\tan\frac{B}{2} = (s-c)\tan\frac{C}{2}}$$

$$\boxed{r = 4R\sin\frac{A}{2}\sin\frac{B}{2}\sin\frac{C}{2}}$$

### Derivation of r = Δ/s

```
    Connect the incenter I to vertices A, B, C.
    This divides triangle ABC into three smaller triangles.
    
              A
             /|\
            / | \
           /  |  \
          / r●I r \
         /  /   \  \
        / r/     \r \
       /  /       \  \
      B───────────────C
    
    Area of △AIB = (1/2) × c × r
    Area of △BIC = (1/2) × a × r
    Area of △CIA = (1/2) × b × r
    
    Total: Δ = (1/2)r(a + b + c) = rs
    
    Therefore: r = Δ/s  ∎
```

---

## 📐 The Incenter Location

The incenter is the intersection of angle bisectors and is always inside the triangle (for all triangle types).

### Coordinates of Incenter

If A(x₁, y₁), B(x₂, y₂), C(x₃, y₃):

$$I = \left(\frac{ax_1 + bx_2 + cx_3}{a+b+c}, \frac{ay_1 + by_2 + cy_3}{a+b+c}\right)$$

---

## 📐 Exradii (Escribed Circle Radii)

Each triangle has three **excircles**, each tangent to one side and the extensions of the other two sides.

```
                    *  *
                  *      *
                 *    r_a  *
                  *   ●    *
                    * | *
              A      \|/
             / \      |
            /   \     |
           /     \    |
          /       \   |
         B─────────C  |
                      |
    
    Excircle opposite to A, with radius r_a
```

### Formulas for Exradii

$$\boxed{r_a = \frac{\Delta}{s-a}}$$

$$\boxed{r_b = \frac{\Delta}{s-b}}$$

$$\boxed{r_c = \frac{\Delta}{s-c}}$$

Also:

$$r_a = s\tan\frac{A}{2}$$

$$r_a = 4R\sin\frac{A}{2}\cos\frac{B}{2}\cos\frac{C}{2}$$

---

## 📐 Important Relationships

### Between r, R, and Triangle Elements

$$\boxed{r = 4R\sin\frac{A}{2}\sin\frac{B}{2}\sin\frac{C}{2}}$$

$$\boxed{r = (s-a)\tan\frac{A}{2}}$$

$$\boxed{r_a + r_b + r_c - r = 4R}$$

$$\boxed{\frac{1}{r_a} + \frac{1}{r_b} + \frac{1}{r_c} = \frac{1}{r}}$$

### Proof of 1/r_a + 1/r_b + 1/r_c = 1/r

```
    1/r_a + 1/r_b + 1/r_c = (s-a)/Δ + (s-b)/Δ + (s-c)/Δ
                         = (3s - (a+b+c))/Δ
                         = (3s - 2s)/Δ
                         = s/Δ
                         = 1/r  ∎
```

---

## 📐 Special Cases

### Equilateral Triangle (side a)

```
    ┌─────────────────────────────────────────────────────────────┐
    │                                                             │
    │   For equilateral triangle with side a:                    │
    │                                                             │
    │   Δ = (√3/4)a²                                              │
    │                                                             │
    │   s = 3a/2                                                  │
    │                                                             │
    │   R = a/√3 = a√3/3                                         │
    │                                                             │
    │   r = a/(2√3) = a√3/6                                      │
    │                                                             │
    │   R/r = 2  (circumradius is twice the inradius)            │
    │                                                             │
    │   Circumcenter = Incenter = Centroid = Orthocenter         │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

### Right Triangle (right angle at C)

```
    R = c/2  (half the hypotenuse)
    
    r = (a + b - c)/2
    
    (since r = Δ/s = (ab/2)/((a+b+c)/2) = ab/(a+b+c)
     and for right triangle: c² = a² + b², so
     r = (a + b - c)/2 can be verified)
```

---

## 📐 Euler's Formula for Distance Between Centers

$$\boxed{OI^2 = R^2 - 2Rr}$$

or equivalently:

$$\boxed{OI = \sqrt{R(R-2r)}}$$

where O is circumcenter, I is incenter.

### Consequence

Since OI² ≥ 0, we have R² ≥ 2Rr, which gives:

$$\boxed{R \geq 2r}$$

with equality only for equilateral triangles.

---

## 📝 Worked Examples

### Example 1: Find R and r for a 3-4-5 Right Triangle

**Solution:**

```
    Sides: a = 3, b = 4, c = 5 (right angle at C)
    
    Semi-perimeter: s = (3 + 4 + 5)/2 = 6
    
    Area: Δ = (1/2)(3)(4) = 6
    
    Circumradius:
    R = abc/(4Δ) = (3 × 4 × 5)/(4 × 6) = 60/24 = 5/2
    
    Or: R = c/2 = 5/2 (for right triangle)
    
    Inradius:
    r = Δ/s = 6/6 = 1
    
    Or: r = (a + b - c)/2 = (3 + 4 - 5)/2 = 1
    
    Verification: R/r = (5/2)/1 = 2.5 > 2 ✓ (as expected for non-equilateral)
```

---

### Example 2: Find All Radii for Triangle with Sides 5, 6, 7

**Solution:**

```
    s = (5 + 6 + 7)/2 = 9
    
    Δ = √[s(s-a)(s-b)(s-c)]
      = √[9 × 4 × 3 × 2]
      = √216 = 6√6
    
    Circumradius:
    R = abc/(4Δ) = (5 × 6 × 7)/(4 × 6√6)
      = 210/(24√6) = 35/(4√6) = 35√6/24
    
    Inradius:
    r = Δ/s = 6√6/9 = 2√6/3
    
    Exradii:
    r_a = Δ/(s-a) = 6√6/(9-5) = 6√6/4 = 3√6/2
    r_b = Δ/(s-b) = 6√6/(9-6) = 6√6/3 = 2√6
    r_c = Δ/(s-c) = 6√6/(9-7) = 6√6/2 = 3√6
    
    Verification of 1/r_a + 1/r_b + 1/r_c = 1/r:
    LHS = 2/(3√6) + 1/(2√6) + 1/(3√6)
        = (4 + 3 + 2)/(6√6)
        = 9/(6√6) = 3/(2√6)
    
    RHS = 1/r = 3/(2√6) ✓
```

---

### Example 3: Proof Problem

**Prove that in any triangle: r = (s-a) tan(A/2)**

**Solution:**

```
    We know r = Δ/s and Δ = (1/2)bc sin A = bc sin(A/2) cos(A/2)
    
    Also, from half-angle formulas:
    tan(A/2) = √[(s-b)(s-c)/(s(s-a))]    ... (1)
    
    And: Δ = √[s(s-a)(s-b)(s-c)]         ... (2)
    
    From (1): tan²(A/2) = (s-b)(s-c)/[s(s-a)]
    
    (s-a) tan(A/2) = (s-a) × √[(s-b)(s-c)/(s(s-a))]
                   = √[(s-a)²(s-b)(s-c)/(s(s-a))]
                   = √[(s-a)(s-b)(s-c)/s]
                   = √[s(s-a)(s-b)(s-c)]/s
                   = Δ/s
                   = r  ∎
```

---

### Example 4: Given r and R, Find Relationship

**In a triangle, r = 2 and R = 5. Find OI (distance between incenter and circumcenter).**

**Solution:**

```
    Using Euler's formula:
    OI² = R² - 2Rr
    OI² = 25 - 2(5)(2)
    OI² = 25 - 20 = 5
    OI = √5
```

---

## 📋 Summary Table

### Key Formulas

| Quantity | Formula 1 | Formula 2 |
|----------|-----------|-----------|
| R (circumradius) | R = a/(2 sin A) | R = abc/(4Δ) |
| r (inradius) | r = Δ/s | r = (s-a) tan(A/2) |
| r_a (exradius) | r_a = Δ/(s-a) | r_a = s tan(A/2) |

### Center Locations

| Center | Definition | Location |
|--------|------------|----------|
| Circumcenter (O) | Intersection of ⊥ bisectors | Depends on triangle type |
| Incenter (I) | Intersection of ∠ bisectors | Always inside |
| Excenter (I_a) | Intersection of external ∠ bisectors | Always outside |

### Important Inequalities and Equalities

| Relationship | Formula |
|--------------|---------|
| Euler's inequality | R ≥ 2r (equality for equilateral) |
| Euler's formula | OI² = R² - 2Rr |
| Sum relation | r_a + r_b + r_c - r = 4R |
| Reciprocal relation | 1/r_a + 1/r_b + 1/r_c = 1/r |

---

## ❓ Quick Revision Questions

1. **State the formula for circumradius in terms of sides and area.**

2. **What is the inradius formula using semi-perimeter and area?**

3. **For a right triangle with legs 6 and 8, find the circumradius and inradius.**

4. **Where is the circumcenter located in an obtuse triangle?**

5. **If R = 10 and r = 3, find the distance between the circumcenter and incenter.**

6. **For an equilateral triangle, what is the ratio R : r?**

<details>
<summary>Click to see answers</summary>

1. **Circumradius formula:**
   $$R = \frac{abc}{4\Delta}$$

2. **Inradius formula:**
   $$r = \frac{\Delta}{s}$$

3. **Right triangle with legs 6 and 8:**
   
   Hypotenuse: c = √(36 + 64) = 10
   
   Area: Δ = (1/2)(6)(8) = 24
   
   Semi-perimeter: s = (6 + 8 + 10)/2 = 12
   
   **R = c/2 = 10/2 = 5**
   
   **r = Δ/s = 24/12 = 2**
   
   (Or: r = (a + b - c)/2 = (6 + 8 - 10)/2 = 2)

4. The circumcenter is located **outside the triangle** for an obtuse triangle, on the opposite side of the longest side from the obtuse angle.

5. **Distance OI with R = 10, r = 3:**
   
   OI² = R² - 2Rr = 100 - 60 = 40
   
   **OI = √40 = 2√10 ≈ 6.32**

6. For an equilateral triangle:
   
   R = a/√3 and r = a/(2√3)
   
   **R : r = 2 : 1**

</details>

---

## Navigation

| Previous | Up | Next Unit |
|----------|-------|-----------|
| [← 8.3 Area Formulas](03-area-formulas.md) | [Unit 8 Index](README.md) | [Unit 9: Heights and Distances →](../09-Heights-and-Distances/README.md) |

---

[← Back to Main Index](../README.md)
