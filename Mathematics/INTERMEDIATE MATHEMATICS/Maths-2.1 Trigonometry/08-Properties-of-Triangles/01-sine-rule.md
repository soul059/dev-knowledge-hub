# Chapter 8.1: The Sine Rule (Law of Sines)

## Overview

The Sine Rule establishes a beautiful relationship between the sides of a triangle and the sines of their opposite angles. It's one of the most powerful tools for solving triangles when we know some angles.

---

## 📐 Statement of the Sine Rule

$$\boxed{\frac{a}{\sin A} = \frac{b}{\sin B} = \frac{c}{\sin C} = 2R}$$

Or equivalently:

$$\boxed{\frac{\sin A}{a} = \frac{\sin B}{b} = \frac{\sin C}{c}}$$

where R is the **circumradius** (radius of the circumscribed circle).

---

## 📝 Proof of the Sine Rule

### Using the Circumcircle

```
    Consider triangle ABC inscribed in a circle with center O
    and radius R.
    
                        A
                       /|\
                      / | \
                     /  |  \
                  c /   |   \ b
                   /    |    \
                  /     |     \
                 /      |      \
                B───────D───────C
                    a
    
    Let D be a point on the circle such that BD is a diameter.
```

**Step 1:** In the circumcircle, angle BDC = angle BAC = A (angles in same segment)

**Step 2:** Since BD is a diameter, angle BCD = 90° (angle in semicircle)

**Step 3:** In right triangle BCD:
$$\sin(\angle BDC) = \sin A = \frac{BC}{BD} = \frac{a}{2R}$$

**Step 4:** Rearranging:
$$\frac{a}{\sin A} = 2R$$

**Step 5:** Similarly for the other sides:
$$\frac{b}{\sin B} = 2R \quad \text{and} \quad \frac{c}{\sin C} = 2R$$

Therefore:
$$\frac{a}{\sin A} = \frac{b}{\sin B} = \frac{c}{\sin C} = 2R \quad \blacksquare$$

---

## 📐 Alternative Proof (Using Area)

```
    The area of a triangle can be expressed as:
    
    Δ = (1/2)bc sin A = (1/2)ca sin B = (1/2)ab sin C
    
    From the first two:
    bc sin A = ca sin B
    b sin A = a sin B
    a/sin A = b/sin B
    
    Similarly for the other pairs.
```

---

## 📐 When to Use the Sine Rule

### Use Sine Rule When You Know:

1. **Two angles and one side** (AAS or ASA)
2. **Two sides and an angle opposite one of them** (SSA - Ambiguous Case)

### Do NOT Use When:

- You have all three sides (use Cosine Rule)
- You have two sides and the included angle (use Cosine Rule)

```
    ┌─────────────────────────────────────────────────────────────┐
    │                                                             │
    │   DECISION CHART: Which Rule to Use?                       │
    │                                                             │
    │   Given:                          Use:                      │
    │   ──────                          ────                      │
    │   Three sides (SSS)        →     Cosine Rule               │
    │   Two sides + included ∠   →     Cosine Rule               │
    │   Two angles + one side    →     Sine Rule                 │
    │   Two sides + opposite ∠   →     Sine Rule (careful!)      │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

---

## 📐 Solving Triangles Using Sine Rule

### Case 1: Given Two Angles and One Side (AAS/ASA)

**Example:** In triangle ABC, A = 45°, B = 60°, and a = 10. Find b and c.

**Solution:**

```
    Step 1: Find the third angle
    C = 180° - A - B = 180° - 45° - 60° = 75°
    
    Step 2: Use Sine Rule to find b
    b/sin B = a/sin A
    b = a · sin B/sin A
    b = 10 · sin 60°/sin 45°
    b = 10 · (√3/2)/(√2/2)
    b = 10 · √3/√2
    b = 10√(3/2) = 5√6 ≈ 12.25
    
    Step 3: Use Sine Rule to find c
    c = a · sin C/sin A
    c = 10 · sin 75°/sin 45°
```

For sin 75°:
```
    sin 75° = sin(45° + 30°)
            = sin 45° cos 30° + cos 45° sin 30°
            = (√2/2)(√3/2) + (√2/2)(1/2)
            = (√6 + √2)/4
    
    c = 10 · [(√6 + √2)/4] / (√2/2)
    c = 10 · (√6 + √2)/4 · 2/√2
    c = 10 · (√6 + √2)/(2√2)
    c = 5(√3 + 1) ≈ 13.66
```

---

### Case 2: The Ambiguous Case (SSA)

When given two sides and an angle opposite to one of them, there may be **0, 1, or 2** possible triangles.

```
    Given: sides a, b and angle A (opposite to side a)
    
    ┌─────────────────────────────────────────────────────────────┐
    │                                                             │
    │   Case Analysis (when A is acute and a < b):               │
    │                                                             │
    │   If a < b sin A: No triangle (side too short)             │
    │   If a = b sin A: One triangle (right angle at B)          │
    │   If b sin A < a < b: Two triangles (ambiguous case)       │
    │   If a ≥ b: One triangle                                   │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

### Visual Representation of the Ambiguous Case

```
    Given: A, b, and a where b sin A < a < b
    
                          * C₁
                         /|
                        / |
                     a /  |a
                      /   |
                     /    |
                    A─────*──────*
                       b     C₂
    
    Two possible triangles: ABC₁ and ABC₂
    
    In this case, angle B has two possible values:
    B₁ and B₂ = 180° - B₁
```

### Example: Ambiguous Case

**In triangle ABC, a = 8, b = 10, and A = 40°. Find the possible values of angle B.**

**Solution:**

```
    Using Sine Rule:
    sin B/b = sin A/a
    sin B = b · sin A/a
    sin B = 10 · sin 40°/8
    sin B = 10 · 0.6428/8
    sin B = 0.8035
    
    Since 0 < sin B < 1, angle B exists.
    
    B₁ = sin⁻¹(0.8035) ≈ 53.5°
    B₂ = 180° - 53.5° = 126.5°
    
    Check if both are valid:
    • For B₁ = 53.5°: A + B₁ = 40° + 53.5° = 93.5° < 180° ✓
    • For B₂ = 126.5°: A + B₂ = 40° + 126.5° = 166.5° < 180° ✓
    
    Both triangles are possible!
```

---

## 📐 Extended Sine Rule Applications

### Finding the Circumradius

From $\frac{a}{\sin A} = 2R$:

$$R = \frac{a}{2\sin A} = \frac{b}{2\sin B} = \frac{c}{2\sin C}$$

**Example:** In triangle ABC, a = 6 and A = 30°. Find R.

```
    R = a/(2 sin A)
    R = 6/(2 · sin 30°)
    R = 6/(2 · 0.5)
    R = 6/1 = 6
```

---

### Finding Sides from R

$$a = 2R\sin A, \quad b = 2R\sin B, \quad c = 2R\sin C$$

---

## 📝 Worked Examples

### Example 1: Complete Triangle Solution

**In triangle ABC, A = 50°, C = 70°, and b = 12. Find all unknown sides and angles.**

**Solution:**

```
    Step 1: Find angle B
    B = 180° - A - C = 180° - 50° - 70° = 60°
    
    Step 2: Use Sine Rule to find a
    a/sin A = b/sin B
    a = b · sin A/sin B
    a = 12 · sin 50°/sin 60°
    a = 12 · 0.766/0.866
    a ≈ 10.61
    
    Step 3: Find c
    c = b · sin C/sin B
    c = 12 · sin 70°/sin 60°
    c = 12 · 0.940/0.866
    c ≈ 13.02
```

**Answer:** B = 60°, a ≈ 10.61, c ≈ 13.02

---

### Example 2: Using Sine Rule for Area

**Find the area of triangle ABC where A = 30°, B = 45°, and c = 10.**

**Solution:**

```
    Step 1: Find angle C
    C = 180° - 30° - 45° = 105°
    
    Step 2: Find sides a and b using Sine Rule
    a/sin A = c/sin C
    a = c · sin A/sin C = 10 · sin 30°/sin 105°
    
    sin 105° = sin(60° + 45°) = (√6 + √2)/4 ≈ 0.966
    
    a = 10 · 0.5/0.966 ≈ 5.18
    
    Similarly: b = c · sin B/sin C = 10 · sin 45°/sin 105°
    b = 10 · 0.707/0.966 ≈ 7.32
    
    Step 3: Calculate area
    Area = (1/2) · a · b · sin C
         = (1/2) · 5.18 · 7.32 · sin 105°
         = (1/2) · 5.18 · 7.32 · 0.966
         ≈ 18.32 square units
```

---

### Example 3: Navigation Problem

**Two lighthouses A and B are 5 km apart on a coastline running east-west. A ship at C observes lighthouse A at a bearing of N40°W and lighthouse B at N70°E. How far is the ship from each lighthouse?**

**Solution:**

```
    Setting up the triangle:
    
                   N
                   ↑
             C *   |
              /\   |
           a /  \ b
            /    \
           /      \
          A────────B
              5 km
    
    Angle ACB = 40° + 70° = 110°
    
    At A: angle CAB = 90° - 40° = 50°  (since A is to the west)
    At B: angle ABC = 90° - 70° = 20°  (since B is to the east)
    
    Check: 50° + 20° + 110° = 180° ✓
    
    Using Sine Rule:
    a/sin 50° = 5/sin 110°
    a = 5 · sin 50°/sin 110°
    a = 5 · 0.766/0.940 ≈ 4.07 km  (distance to B)
    
    b/sin 20° = 5/sin 110°
    b = 5 · sin 20°/sin 110°
    b = 5 · 0.342/0.940 ≈ 1.82 km  (distance to A)
```

---

## 📋 Summary Table

### Sine Rule Forms

| Form | Formula | Use |
|------|---------|-----|
| Standard | a/sin A = b/sin B = c/sin C | Finding sides |
| Reciprocal | sin A/a = sin B/b = sin C/c | Finding angles |
| With circumradius | a/sin A = 2R | Finding R |

### Ambiguous Case Summary (Given a, b, A where A is acute)

| Condition | Number of Triangles |
|-----------|---------------------|
| a < b sin A | 0 (impossible) |
| a = b sin A | 1 (right triangle) |
| b sin A < a < b | 2 (ambiguous) |
| a ≥ b | 1 |

---

## ❓ Quick Revision Questions

1. **State the Sine Rule.**

2. **In triangle ABC, A = 30°, B = 45°, and a = 8. Find b.**

3. **What is the relationship between the Sine Rule and the circumradius?**

4. **In triangle ABC, a = 7, b = 8, and A = 35°. Determine if there are 0, 1, or 2 possible triangles.**

5. **If a = 2R sin A, find the value of sin A + sin B + sin C in terms of a, b, c, and R.**

6. **A triangle has circumradius R = 5 and angle A = 60°. Find side a.**

<details>
<summary>Click to see answers</summary>

1. **Sine Rule:**
   $$\frac{a}{\sin A} = \frac{b}{\sin B} = \frac{c}{\sin C} = 2R$$

2. **Finding b:**
   b/sin B = a/sin A
   b = a · sin B/sin A
   b = 8 · sin 45°/sin 30°
   b = 8 · (√2/2)/(1/2)
   b = 8 · √2 = **8√2 ≈ 11.31**

3. Each ratio in the Sine Rule equals **2R**, where R is the circumradius:
   $$\frac{a}{\sin A} = 2R$$

4. **Checking ambiguity:**
   b sin A = 8 · sin 35° = 8 · 0.574 ≈ 4.59
   
   Since b sin A (4.59) < a (7) < b (8), there are **2 possible triangles**.

5. From a = 2R sin A, we have sin A = a/(2R)
   Similarly: sin B = b/(2R), sin C = c/(2R)
   
   **sin A + sin B + sin C = (a + b + c)/(2R)**

6. a = 2R sin A = 2 · 5 · sin 60° = 10 · (√3/2) = **5√3 ≈ 8.66**

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Unit 7: Inverse Trig Functions](../07-Inverse-Trigonometric-Functions/README.md) | [Unit 8 Index](README.md) | [8.2 Cosine Rule →](02-cosine-rule.md) |

---

[← Back to Main Index](../README.md)
