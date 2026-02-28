# Chapter 1.1: Angle Measurement

## Overview

Understanding how angles are measured is the foundation of trigonometry. This chapter covers the two primary systems of angle measurement: **degrees** and **radians**, along with conversions between them.

---

## 📐 What is an Angle?

An **angle** is formed by two rays (called the sides of the angle) sharing a common endpoint (called the **vertex**). The amount of rotation from one ray to another determines the measure of the angle.

```
                    Terminal Side
                   /
                  /
                 /
                /
               /  θ (angle)
              /__________ Initial Side
             O (Vertex)
```

### Types of Angle Rotation

- **Positive Angle**: Counter-clockwise rotation
- **Negative Angle**: Clockwise rotation

```
    Positive Rotation              Negative Rotation
    (Counter-clockwise)            (Clockwise)
           ↺                              ↻
          /                                \
         /                                  \
        / +θ                            -θ   \
       /________                    ________\
```

---

## 📏 Degree Measurement

### Definition

A **degree** (°) is defined as 1/360th of a complete rotation. One complete rotation = 360°.

### Historical Note

The choice of 360 comes from ancient Babylonians who used a base-60 number system. 360 is convenient because it has many divisors (1, 2, 3, 4, 5, 6, 8, 9, 10, 12, 15, 18, 20, 24, 30, 36, 40, 45, 60, 72, 90, 120, 180, 360).

### Subdivisions of Degrees

| Unit | Symbol | Relation |
|------|--------|----------|
| Degree | ° | Base unit |
| Minute | ' | 1° = 60' |
| Second | " | 1' = 60" = 1° = 3600" |

### Types of Angles by Degree Measure

```
    Acute Angle         Right Angle        Obtuse Angle
    (0° < θ < 90°)      (θ = 90°)          (90° < θ < 180°)
    
         /                   |                    /
        /                    |                   /
       / θ                   |                  /  θ
      /____                  |____             /____
      
    Straight Angle      Reflex Angle       Full Rotation
    (θ = 180°)          (180° < θ < 360°)  (θ = 360°)
    
    __________               ____              ____
                            /    \            /    \
                            \  θ /            |  θ  |
                             \__/             \____/
```

---

## 🔄 Radian Measurement

### Definition

A **radian** is the angle subtended at the center of a circle by an arc whose length equals the radius of the circle.

```
                     Arc length = r
                    ___________
                   /           \
                  /             \
                 |       r       |
                 |      /        |
                 |     /         |
                 |    / θ = 1 rad|
                 |   /           |
                  \ /___________/
                   \_____r_____/
                        
    When arc length = radius, θ = 1 radian
```

### Mathematical Definition

$$\theta \text{ (in radians)} = \frac{\text{Arc Length}}{\text{Radius}} = \frac{s}{r}$$

### Key Radian Values

Since the circumference of a circle = 2πr:
- **Full rotation** = 2πr/r = **2π radians**
- **Half rotation** = **π radians**
- **Quarter rotation** = **π/2 radians**

---

## 🔁 Degree-Radian Conversion

### Fundamental Relationship

$$360° = 2\pi \text{ radians}$$
$$180° = \pi \text{ radians}$$

### Conversion Formulas

**Degrees to Radians:**
$$\text{Radians} = \text{Degrees} \times \frac{\pi}{180°}$$

**Radians to Degrees:**
$$\text{Degrees} = \text{Radians} \times \frac{180°}{\pi}$$

### Common Angle Conversions

| Degrees | Radians | Decimal (approx) |
|---------|---------|------------------|
| 0° | 0 | 0 |
| 30° | π/6 | 0.524 |
| 45° | π/4 | 0.785 |
| 60° | π/3 | 1.047 |
| 90° | π/2 | 1.571 |
| 120° | 2π/3 | 2.094 |
| 135° | 3π/4 | 2.356 |
| 150° | 5π/6 | 2.618 |
| 180° | π | 3.142 |
| 270° | 3π/2 | 4.712 |
| 360° | 2π | 6.283 |

---

## 📊 Visual Angle Reference

```
                            90° (π/2)
                               |
                               |
                    120°       |       60°
                   (2π/3) \    |    / (π/3)
                           \   |   /
                  135°      \  |  /      45°
                 (3π/4) -----\-|-/------ (π/4)
                              \|/
        180° (π) ──────────────●────────────── 0° (0)
                              /|\              360° (2π)
                 (5π/4) -----/-|-\------ (7π/4)
                  225°      /  |  \      315°
                           /   |   \
                    240°  /    |    \ 300°
                   (4π/3)      |      (5π/3)
                               |
                            270° (3π/2)
```

---

## 🧮 Worked Examples

### Example 1: Convert 150° to radians

**Solution:**
$$\text{Radians} = 150° \times \frac{\pi}{180°} = \frac{150\pi}{180} = \frac{5\pi}{6} \text{ radians}$$

### Example 2: Convert 5π/4 radians to degrees

**Solution:**
$$\text{Degrees} = \frac{5\pi}{4} \times \frac{180°}{\pi} = \frac{5 \times 180°}{4} = \frac{900°}{4} = 225°$$

### Example 3: Express 72°30' in radians

**Solution:**
First, convert to decimal degrees:
$$72°30' = 72° + \frac{30}{60}° = 72.5°$$

Then convert to radians:
$$72.5° \times \frac{\pi}{180°} = \frac{72.5\pi}{180} = \frac{145\pi}{360} = \frac{29\pi}{72} \text{ radians}$$

### Example 4: Find the arc length

A circle has radius 10 cm. Find the arc length subtending an angle of π/3 radians at the center.

**Solution:**
Using $s = r\theta$:
$$s = 10 \times \frac{\pi}{3} = \frac{10\pi}{3} \approx 10.47 \text{ cm}$$

---

## 🌍 Real-World Applications

### 1. Navigation and Bearings
Ships and aircraft use degree measurements for navigation. A bearing of 045° means 45° clockwise from North.

### 2. Engineering and Physics
Radians are preferred in calculus and physics because they simplify many formulas. Angular velocity is measured in rad/s.

### 3. Computer Graphics
Rotation transformations in graphics programming use radians for mathematical convenience.

### 4. Astronomy
Celestial coordinates use degrees, minutes, and seconds to specify positions of stars and planets.

---

## 📋 Summary Table

| Concept | Formula/Value |
|---------|---------------|
| Full rotation | 360° = 2π rad |
| Half rotation | 180° = π rad |
| Right angle | 90° = π/2 rad |
| Degrees to radians | θ(rad) = θ(deg) × π/180 |
| Radians to degrees | θ(deg) = θ(rad) × 180/π |
| Arc length formula | s = rθ (θ in radians) |
| 1 radian | ≈ 57.2958° |
| 1 degree | ≈ 0.01745 rad |

---

## ❓ Quick Revision Questions

1. **Convert 240° to radians.** Express your answer in terms of π.

2. **Convert 7π/6 radians to degrees.**

3. **Why is radian measure preferred in calculus over degrees?**

4. **An arc of length 15 cm subtends an angle at the center of a circle of radius 9 cm. Find the angle in radians and degrees.**

5. **Express 45°15'30" in decimal degrees.**

6. **The minute hand of a clock is 14 cm long. How far does the tip of the minute hand move in 20 minutes?**

<details>
<summary>Click to see answers</summary>

1. 240° = 240 × π/180 = **4π/3 radians**

2. 7π/6 × 180/π = 7 × 30 = **210°**

3. Radians simplify derivative formulas. For example, d/dx(sin x) = cos x only when x is in radians.

4. θ = s/r = 15/9 = **5/3 radians** ≈ 1.667 rad = 1.667 × 180/π ≈ **95.5°**

5. 45° + 15/60 + 30/3600 = 45° + 0.25° + 0.00833° = **45.2583°**

6. In 20 minutes, minute hand rotates 20/60 × 360° = 120° = 2π/3 rad.  
   Arc length = rθ = 14 × 2π/3 = **28π/3 ≈ 29.32 cm**

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| - | [Unit 1 Index](README.md) | [1.2 Trigonometric Ratios →](02-trigonometric-ratios.md) |

---

[← Back to Main Index](../README.md)
