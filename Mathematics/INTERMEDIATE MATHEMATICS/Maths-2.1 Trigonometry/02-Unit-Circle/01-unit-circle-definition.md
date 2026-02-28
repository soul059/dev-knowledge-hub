# Chapter 2.1: Unit Circle Definition

## Overview

The unit circle is a fundamental concept that allows us to extend trigonometric functions beyond the limitations of right triangles. By defining trigonometric ratios using coordinates on a circle, we can work with any angle—positive, negative, or greater than 360°.

---

## 📐 What is the Unit Circle?

### Definition

The **unit circle** is a circle with:
- **Center** at the origin (0, 0)
- **Radius** of exactly 1 unit

### Equation

$$x^2 + y^2 = 1$$

```
                           y
                           │
                      1    │    
                    ╱──────┼──────╲
                  ╱        │        ╲
                ╱          │          ╲
              ╱            │            ╲
             │             │             │
        ─────┼─────────────●─────────────┼───── x
            -1             │             1
             │             │             │
              ╲            │            ╱
                ╲          │          ╱
                  ╲        │        ╱
                    ╲──────┼──────╱
                          -1
                           │
                           
    Center: (0, 0)
    Radius: 1
    Equation: x² + y² = 1
```

---

## 📍 Points on the Unit Circle

### Coordinate Representation

Any point P on the unit circle can be described by its coordinates (x, y), where:
- x = horizontal distance from origin
- y = vertical distance from origin

Since the radius is 1: **x² + y² = 1**

### Angle and Coordinates

For an angle θ measured counter-clockwise from the positive x-axis:

```
                           y
                           │
                           │    P(x, y) = P(cos θ, sin θ)
                           │   ╱
                      1    │  ╱
                    ╱──────┼─●──────╲
                  ╱        │╱|       ╲
                ╱          ╱ │         ╲
              ╱           ╱  │           ╲
             │           ╱ θ │ y = sin θ  │
        ─────┼──────────●────┼────────────┼───── x
            -1          │    │            1
             │          └────┘             │
              ╲         x = cos θ         ╱
                ╲          │          ╱
                  ╲        │        ╱
                    ╲──────┼──────╱
                          -1
                           │
```

### The Fundamental Definition

For any angle θ, if P(x, y) is the point on the unit circle:

$$\boxed{\cos \theta = x \quad \text{and} \quad \sin \theta = y}$$

This means:
- **Cosine** is the **x-coordinate** of the point
- **Sine** is the **y-coordinate** of the point

---

## 🔗 Connection to Right Triangle Definitions

Consider the right triangle formed by dropping a perpendicular from point P to the x-axis:

```
                    P(x, y)
                   /|
                  / |
            r=1  /  | y (opposite)
                /   |
               / θ  |
              /_____|____
              O     x    
              (adjacent)
```

From the right triangle definition:
$$\sin \theta = \frac{\text{opposite}}{\text{hypotenuse}} = \frac{y}{1} = y$$

$$\cos \theta = \frac{\text{adjacent}}{\text{hypotenuse}} = \frac{x}{1} = x$$

**This proves that the unit circle definition is consistent with the right triangle definition!**

---

## 📊 Key Points on the Unit Circle

### Cardinal Points (Quadrantal Angles)

| Angle | Radians | Point (x, y) | cos θ | sin θ |
|-------|---------|--------------|-------|-------|
| 0° | 0 | (1, 0) | 1 | 0 |
| 90° | π/2 | (0, 1) | 0 | 1 |
| 180° | π | (-1, 0) | -1 | 0 |
| 270° | 3π/2 | (0, -1) | 0 | -1 |
| 360° | 2π | (1, 0) | 1 | 0 |

```
                         (0, 1)
                           90°
                           │
                           │
                           │
     (-1, 0) ──────────────●────────────── (1, 0)
       180°                │                 0°/360°
                           │
                           │
                           │
                         (0, -1)
                          270°
```

### Special Angle Points (First Quadrant)

| Angle | Radians | Point (cos θ, sin θ) |
|-------|---------|----------------------|
| 30° | π/6 | (√3/2, 1/2) |
| 45° | π/4 | (√2/2, √2/2) |
| 60° | π/3 | (1/2, √3/2) |

---

## 🎯 Complete Unit Circle Diagram

```
                              90° (π/2)
                               (0, 1)
                                 │
              120° (2π/3)        │        60° (π/3)
             (-1/2, √3/2)   \    │    /   (1/2, √3/2)
                             \   │   /
           135° (3π/4)        \  │  /      45° (π/4)
          (-√2/2, √2/2)  ------\-│-/------ (√2/2, √2/2)
                                \│/
        150° (5π/6)              │              30° (π/6)
       (-√3/2, 1/2)    ─────────●─────────   (√3/2, 1/2)
                                │
   180° (π)  ───────────────────●─────────────────── 0° (0)
   (-1, 0)                      │                   (1, 0)
                                │
       210° (7π/6)              │              330° (11π/6)
       (-√3/2, -1/2)  ─────────●─────────  (√3/2, -1/2)
                               /│\
          225° (5π/4)        /  │  \       315° (7π/4)
         (-√2/2, -√2/2) ----/--│--\---- (√2/2, -√2/2)
                            /   │   \
             240° (4π/3)   /    │    \   300° (5π/3)
            (-1/2, -√3/2)       │       (1/2, -√3/2)
                                │
                              (0, -1)
                             270° (3π/2)
```

---

## 📏 Other Trigonometric Functions on the Unit Circle

### Tangent as a Ratio

$$\tan \theta = \frac{\sin \theta}{\cos \theta} = \frac{y}{x}$$

### Geometric Interpretation of Tangent

The tangent can be visualized as the length of the tangent line segment from the x-axis:

```
                       │
                       │   T (tangent point)
                       │  /
                       │ / tan θ
                       │/
            ───────────●─────────────
                      /│ θ
                     / │
                    /  │
                   ●   │
                   P   │
                       │
                       
    tan θ = length from (1, 0) to T along the vertical tangent line
```

### All Six Functions

| Function | Unit Circle Definition |
|----------|----------------------|
| sin θ | y-coordinate of P |
| cos θ | x-coordinate of P |
| tan θ | y/x = sin θ/cos θ |
| cot θ | x/y = cos θ/sin θ |
| sec θ | 1/x = 1/cos θ |
| csc θ | 1/y = 1/sin θ |

---

## 🔄 Understanding Angles on the Unit Circle

### Positive vs Negative Angles

```
    Positive Angle               Negative Angle
    (Counter-clockwise)          (Clockwise)
    
           ↺                          ↻
          /                            \
         / +θ                      -θ   \
        /                                \
    ───●───────────              ───●───────────
      O                            O
```

### Coterminal Angles

Angles that differ by full rotations (360° or 2π) end at the same point.

$$\theta \text{ and } \theta + 360°n \text{ are coterminal}$$

**Examples:**
- 45° and 405° are coterminal (405° = 45° + 360°)
- 30° and -330° are coterminal (-330° = 30° - 360°)

```
    45° and 405° both end here:
    
              ╱
             ╱
            ●  P
           ╱│
          ╱ │
    ─────●──┴─────
         O
```

---

## 🧮 Worked Examples

### Example 1: Finding Coordinates

Find the coordinates of the point on the unit circle corresponding to θ = 150°.

**Solution:**

150° is in the second quadrant.
Reference angle = 180° - 150° = 30°

For 30°: (√3/2, 1/2)

In Quadrant II: x is negative, y is positive

Therefore, point for 150° is **(-√3/2, 1/2)**

So: cos 150° = -√3/2, sin 150° = 1/2

### Example 2: Verifying a Point

Verify that the point (3/5, 4/5) lies on the unit circle, and find the angle.

**Solution:**

Check: x² + y² = (3/5)² + (4/5)² = 9/25 + 16/25 = 25/25 = 1 ✓

The point is on the unit circle.

sin θ = 4/5 = 0.8, cos θ = 3/5 = 0.6

θ = arcsin(0.8) ≈ **53.13°**

### Example 3: Finding All Six Ratios

Given a point P(-5/13, 12/13) on a circle centered at origin, find all six trigonometric ratios.

**Solution:**

First verify: (-5/13)² + (12/13)² = 25/169 + 144/169 = 169/169 = 1 ✓

| Ratio | Value |
|-------|-------|
| sin θ | y = 12/13 |
| cos θ | x = -5/13 |
| tan θ | y/x = 12/(-5) = -12/5 |
| cot θ | x/y = -5/12 |
| sec θ | 1/x = -13/5 |
| csc θ | 1/y = 13/12 |

### Example 4: Coterminal Angles

Find a coterminal angle between 0° and 360° for θ = -735°.

**Solution:**

-735° + 360° = -375° (still negative)
-375° + 360° = -15° (still negative)
-15° + 360° = **345°**

Or: -735° = -735° + 3(360°) = -735° + 1080° = **345°**

---

## 🔍 Why is the Unit Circle Useful?

### 1. Extends Beyond Right Triangles
- Right triangles only work for 0° < θ < 90°
- Unit circle works for ANY angle

### 2. Simplifies Calculations
- Radius = 1 eliminates division
- Direct reading of sine and cosine from coordinates

### 3. Reveals Periodicity
- Same point reached every 360°
- Shows why trig functions repeat

### 4. Geometric Insight
- Visual understanding of function behavior
- Clear relationship between angle and coordinates

---

## 🌍 Real-World Applications

### 1. Circular Motion
Objects moving in circles (wheels, planets) have positions described by unit circle coordinates.

### 2. Waves and Oscillations
Sound waves, light waves, and electrical signals use sine/cosine functions derived from unit circle.

### 3. Computer Graphics
Rotation transformations in 2D and 3D graphics use unit circle principles.

### 4. Navigation
Direction vectors and bearings are naturally expressed using unit circle coordinates.

---

## 📋 Summary Table

| Concept | Definition/Formula |
|---------|---------------------|
| Unit circle | Circle with center (0,0), radius 1 |
| Equation | x² + y² = 1 |
| sin θ | y-coordinate of point P |
| cos θ | x-coordinate of point P |
| Point P | (cos θ, sin θ) |
| tan θ | y/x = sin θ/cos θ |
| Coterminal angles | Angles differing by 360°n |

### Key Points on Unit Circle

| Angle | Coordinates |
|-------|-------------|
| 0° | (1, 0) |
| 30° | (√3/2, 1/2) |
| 45° | (√2/2, √2/2) |
| 60° | (1/2, √3/2) |
| 90° | (0, 1) |
| 180° | (-1, 0) |
| 270° | (0, -1) |

---

## ❓ Quick Revision Questions

1. **What is the equation of the unit circle?**

2. **If a point on the unit circle has coordinates (0.6, 0.8), what are sin θ and cos θ?**

3. **Why can sin θ and cos θ never exceed 1 in absolute value on the unit circle?**

4. **Find two angles coterminal with 60°, one positive and one negative.**

5. **The point (-√2/2, -√2/2) is on the unit circle. What angle does it correspond to?**

6. **Explain why tan 90° is undefined using the unit circle definition.**

<details>
<summary>Click to see answers</summary>

1. **x² + y² = 1**

2. cos θ = x = **0.6**, sin θ = y = **0.8**

3. Because all points on the unit circle are at distance 1 from the origin. The coordinates (x, y) satisfy x² + y² = 1, so neither x nor y can exceed 1 in absolute value.

4. Positive: 60° + 360° = **420°**  
   Negative: 60° - 360° = **-300°**

5. Both coordinates are negative, so Quadrant III. Reference angle with |√2/2| values is 45°.  
   Angle = 180° + 45° = **225°** (or 5π/4)

6. At 90°, the point is (0, 1). tan 90° = y/x = 1/0, which is **undefined** (division by zero).

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 1.4 Trig Tables](../01-Introduction-to-Trigonometry/04-trigonometric-tables.md) | [Unit 2 Index](README.md) | [2.2 Signs in Quadrants →](02-signs-in-quadrants.md) |

---

[← Back to Main Index](../README.md)
