# Chapter 8.2: The Cosine Rule (Law of Cosines)

## Overview

The Cosine Rule is a generalization of the Pythagorean theorem that works for any triangle. It relates the sides of a triangle to the cosine of one of its angles and is essential when the Sine Rule cannot be applied.

---

## 📐 Statement of the Cosine Rule

$$\boxed{a^2 = b^2 + c^2 - 2bc\cos A}$$

$$\boxed{b^2 = c^2 + a^2 - 2ca\cos B}$$

$$\boxed{c^2 = a^2 + b^2 - 2ab\cos C}$$

### Alternative Form (Finding Angles)

$$\boxed{\cos A = \frac{b^2 + c^2 - a^2}{2bc}}$$

$$\boxed{\cos B = \frac{c^2 + a^2 - b^2}{2ca}}$$

$$\boxed{\cos C = \frac{a^2 + b^2 - c^2}{2ab}}$$

---

## 📝 Proof of the Cosine Rule

### Method 1: Using Coordinates

```
    Place vertex A at the origin with side c along the x-axis:
    
         y
         │
         │        C (b cos A, b sin A)
         │       /
         │      /
         │     / b
         │    /
         │   /
         │  /
         │ /
         │/ A
    ─────*───────────*────── x
        (0,0)      B (c, 0)
```

**Coordinates:**
- A = (0, 0)
- B = (c, 0)
- C = (b cos A, b sin A)

**Finding side a (distance from B to C):**

$$a^2 = (b\cos A - c)^2 + (b\sin A - 0)^2$$

$$a^2 = b^2\cos^2 A - 2bc\cos A + c^2 + b^2\sin^2 A$$

$$a^2 = b^2(\cos^2 A + \sin^2 A) + c^2 - 2bc\cos A$$

$$a^2 = b^2 + c^2 - 2bc\cos A \quad \blacksquare$$

---

### Method 2: Using the Projection Formula

```
    Draw altitude from C to side c:
    
              C
             /|\
            / | \
         b /  |h \ a
          /   |   \
         /    |    \
        A─────D─────B
           x    c-x
        
    Let AD = x, then DB = c - x
```

**In right triangle ACD:**
$$x = b\cos A, \quad h = b\sin A$$

**In right triangle BCD:**
$$a^2 = h^2 + (c-x)^2$$

$$a^2 = b^2\sin^2 A + (c - b\cos A)^2$$

$$a^2 = b^2\sin^2 A + c^2 - 2bc\cos A + b^2\cos^2 A$$

$$a^2 = b^2(\sin^2 A + \cos^2 A) + c^2 - 2bc\cos A$$

$$a^2 = b^2 + c^2 - 2bc\cos A \quad \blacksquare$$

---

## 📐 Connection to Pythagorean Theorem

When angle A = 90°:

$$a^2 = b^2 + c^2 - 2bc\cos 90°$$

$$a^2 = b^2 + c^2 - 2bc(0)$$

$$a^2 = b^2 + c^2$$

This is the Pythagorean theorem! The Cosine Rule is a generalization.

```
    ┌─────────────────────────────────────────────────────────────┐
    │                                                             │
    │   Relationship to Pythagorean Theorem:                     │
    │                                                             │
    │   a² = b² + c² - 2bc cos A                                 │
    │                    ↑                                        │
    │            "correction term"                                │
    │                                                             │
    │   If A < 90°: cos A > 0, so a² < b² + c² (acute)           │
    │   If A = 90°: cos A = 0, so a² = b² + c² (right)           │
    │   If A > 90°: cos A < 0, so a² > b² + c² (obtuse)          │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

---

## 📐 When to Use the Cosine Rule

### Use Cosine Rule When You Know:

1. **All three sides (SSS)** → to find any angle
2. **Two sides and the included angle (SAS)** → to find the third side

```
    ┌─────────────────────────────────────────────────────────────┐
    │                                                             │
    │   SSS Case:                     SAS Case:                   │
    │                                                             │
    │        C                            C                       │
    │       /\                           /\                       │
    │    b /  \ a         Given:      b /  \                      │
    │     /    \          b, c, A      /    \                     │
    │    /      \                     /A     \                    │
    │   A────────B                   A────────B                   │
    │        c                            c                       │
    │                                                             │
    │   Know: a, b, c                 Know: b, c, A               │
    │   Find: A, B, C                 Find: a                     │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

---

## 📝 Worked Examples

### Example 1: Finding a Side (SAS)

**In triangle ABC, b = 7, c = 5, and A = 60°. Find side a.**

**Solution:**

```
    Using: a² = b² + c² - 2bc cos A
    
    a² = 7² + 5² - 2(7)(5) cos 60°
    a² = 49 + 25 - 70(0.5)
    a² = 74 - 35
    a² = 39
    a = √39 ≈ 6.24
```

**Answer: a = √39 ≈ 6.24**

---

### Example 2: Finding an Angle (SSS)

**In triangle ABC, a = 8, b = 6, c = 5. Find angle A.**

**Solution:**

```
    Using: cos A = (b² + c² - a²)/(2bc)
    
    cos A = (6² + 5² - 8²)/(2 · 6 · 5)
    cos A = (36 + 25 - 64)/60
    cos A = -3/60
    cos A = -0.05
    
    A = cos⁻¹(-0.05) ≈ 92.87°
```

**Answer: A ≈ 92.87° (obtuse angle)**

---

### Example 3: Finding All Angles (SSS)

**In triangle ABC, a = 7, b = 8, c = 9. Find all angles.**

**Solution:**

```
    Finding angle A:
    cos A = (b² + c² - a²)/(2bc)
    cos A = (64 + 81 - 49)/(2 · 8 · 9)
    cos A = 96/144 = 2/3
    A = cos⁻¹(2/3) ≈ 48.19°
    
    Finding angle B:
    cos B = (c² + a² - b²)/(2ca)
    cos B = (81 + 49 - 64)/(2 · 9 · 7)
    cos B = 66/126 = 11/21
    B = cos⁻¹(11/21) ≈ 58.41°
    
    Finding angle C:
    C = 180° - A - B
    C = 180° - 48.19° - 58.41° ≈ 73.40°
    
    Verification using Cosine Rule:
    cos C = (a² + b² - c²)/(2ab)
    cos C = (49 + 64 - 81)/(2 · 7 · 8)
    cos C = 32/112 = 2/7
    C = cos⁻¹(2/7) ≈ 73.40° ✓
```

---

### Example 4: Checking Triangle Type

**Determine whether a triangle with sides 5, 7, 9 is acute, right, or obtuse.**

**Solution:**

```
    The largest angle is opposite the largest side.
    Let c = 9 (largest side), so we check angle C.
    
    cos C = (a² + b² - c²)/(2ab)
    cos C = (25 + 49 - 81)/(2 · 5 · 7)
    cos C = -7/70 = -0.1
    
    Since cos C < 0, angle C > 90°
    
    The triangle is OBTUSE.
```

**Quick Method:**
- If a² + b² > c², triangle is acute
- If a² + b² = c², triangle is right
- If a² + b² < c², triangle is obtuse

Here: 25 + 49 = 74 < 81 = c², so obtuse.

---

## 📐 The Projection Formula

A useful corollary of the Cosine Rule:

$$\boxed{a = b\cos C + c\cos B}$$

$$\boxed{b = c\cos A + a\cos C}$$

$$\boxed{c = a\cos B + b\cos A}$$

### Proof

```
    From the figure with altitude from C:
    
              C
             /|\
            / | \
         b /  |h \ a
          /   |   \
         /    |    \
        A─────D─────B
           x    y
    
    AD = x = b cos A
    DB = y = a cos B
    
    c = AD + DB = b cos A + a cos B ∎
```

---

## 📐 Applications of Cosine Rule

### Distance Between Two Points

**Problem:** Two ships leave port at the same time. Ship A travels at 20 km/h on bearing N30°E. Ship B travels at 25 km/h on bearing S40°E. How far apart are they after 2 hours?

```
                   N
                   ↑
                   │  A (after 2 hrs)
                   │ /
                   │/ 30°
         Port *────────────
                   |\40°
                   │ \
                   │  B (after 2 hrs)
    
    Distance traveled:
    Ship A: 20 × 2 = 40 km
    Ship B: 25 × 2 = 50 km
    
    Angle between paths: 30° + 40° = 70°
```

**Solution:**

```
    Using Cosine Rule:
    d² = 40² + 50² - 2(40)(50) cos 70°
    d² = 1600 + 2500 - 4000(0.342)
    d² = 4100 - 1368
    d² = 2732
    d ≈ 52.27 km
```

---

### Finding Diagonals of a Parallelogram

**In parallelogram ABCD, AB = 8, BC = 6, and angle ABC = 60°. Find the diagonals.**

```
        D────────────C
       /            /
      /            /
     /   60°      /
    A────────────B
         8
```

**Solution:**

```
    Diagonal AC (using triangle ABC):
    AC² = AB² + BC² - 2(AB)(BC) cos 60°
    AC² = 64 + 36 - 96(0.5)
    AC² = 100 - 48 = 52
    AC = √52 = 2√13 ≈ 7.21
    
    Diagonal BD (angle ABD = 180° - 60° = 120°):
    BD² = AB² + AD² - 2(AB)(AD) cos 120°
    BD² = 64 + 36 - 96(-0.5)
    BD² = 100 + 48 = 148
    BD = √148 = 2√37 ≈ 12.17
```

---

## 📐 Stewart's Theorem (Advanced Application)

For a cevian AD of length d in triangle ABC, where BD = m and DC = n:

$$b^2m + c^2n = a(d^2 + mn)$$

```
            A
           /|\
          / | \
       c /  |d \ b
        /   |   \
       /    |    \
      B──m──D──n──C
            a
```

This can be proved using the Cosine Rule twice.

---

## 📋 Summary Table

### Cosine Rule Forms

| Purpose | Formula | When to Use |
|---------|---------|-------------|
| Find side | a² = b² + c² - 2bc cos A | Given SAS |
| Find angle | cos A = (b² + c² - a²)/(2bc) | Given SSS |

### Determining Triangle Type

| Condition | Triangle Type |
|-----------|---------------|
| a² + b² > c² | Acute (if c is longest) |
| a² + b² = c² | Right (angle at C) |
| a² + b² < c² | Obtuse (angle C > 90°) |

### Comparison with Sine Rule

| Sine Rule | Cosine Rule |
|-----------|-------------|
| a/sin A = b/sin B | a² = b² + c² - 2bc cos A |
| Use for AAS, ASA, SSA | Use for SAS, SSS |
| May have ambiguous case | Always gives unique solution |
| Related to circumradius | Generalizes Pythagorean theorem |

---

## ❓ Quick Revision Questions

1. **State the Cosine Rule for finding side a.**

2. **In triangle ABC, b = 5, c = 8, and A = 120°. Find a.**

3. **A triangle has sides 5, 12, 13. Is it acute, right, or obtuse?**

4. **In triangle ABC, a = 6, b = 7, c = 8. Find cos C.**

5. **What happens to the Cosine Rule when angle A = 90°?**

6. **Two forces of 10N and 15N act at an angle of 60° to each other. Find the magnitude of the resultant force.**

<details>
<summary>Click to see answers</summary>

1. **Cosine Rule for side a:**
   $$a^2 = b^2 + c^2 - 2bc\cos A$$

2. **Finding a with b = 5, c = 8, A = 120°:**
   
   a² = 5² + 8² - 2(5)(8) cos 120°
   a² = 25 + 64 - 80(-0.5)
   a² = 89 + 40 = 129
   **a = √129 ≈ 11.36**

3. **Triangle with sides 5, 12, 13:**
   
   Check: 5² + 12² = 25 + 144 = 169 = 13²
   
   Since a² + b² = c², it's a **right triangle**.

4. **Finding cos C with a = 6, b = 7, c = 8:**
   
   cos C = (a² + b² - c²)/(2ab)
   cos C = (36 + 49 - 64)/(2 · 6 · 7)
   cos C = 21/84 = **1/4 = 0.25**

5. When A = 90°, cos A = 0, so:
   a² = b² + c² - 2bc(0) = b² + c²
   
   This is the **Pythagorean theorem**.

6. **Resultant of two forces at 60°:**
   
   Using the parallelogram law (Cosine Rule):
   R² = 10² + 15² + 2(10)(15) cos 60°
   
   Note: For resultant of forces, we use + not - because
   the angle in the force triangle is 180° - 60° = 120°
   
   Alternatively: R² = 10² + 15² - 2(10)(15) cos 120°
   R² = 100 + 225 - 300(-0.5)
   R² = 325 + 150 = 475
   **R = √475 ≈ 21.79 N**

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 8.1 Sine Rule](01-sine-rule.md) | [Unit 8 Index](README.md) | [8.3 Area Formulas →](03-area-formulas.md) |

---

[← Back to Main Index](../README.md)
