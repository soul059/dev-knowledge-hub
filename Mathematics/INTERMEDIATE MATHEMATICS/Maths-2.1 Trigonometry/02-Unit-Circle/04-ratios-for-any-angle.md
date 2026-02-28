# Chapter 2.4: Trigonometric Ratios for Any Angle

## Overview

This chapter brings together all the concepts from Unit 2 to show how to find trigonometric ratios for **any angle**—positive or negative, large or small. By combining the unit circle, ASTC rule, and reference angles, we can evaluate any trigonometric expression.

---

## 📐 The Complete Method

### Three-Step Process

```
    ┌─────────────────────────────────────────────────────────────┐
    │     FINDING TRIG RATIOS FOR ANY ANGLE                       │
    ├─────────────────────────────────────────────────────────────┤
    │                                                             │
    │  Step 1: NORMALIZE the angle                                │
    │          • For θ > 360°: subtract 360° until 0° ≤ θ < 360° │
    │          • For θ < 0°: add 360° until 0° ≤ θ < 360°        │
    │                                                             │
    │  Step 2: FIND the reference angle                           │
    │          • Identify the quadrant                            │
    │          • Apply appropriate formula                        │
    │                                                             │
    │  Step 3: DETERMINE the sign and calculate                   │
    │          • Use ASTC rule for the quadrant                   │
    │          • Apply sign to the reference angle value          │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

---

## 🔄 Handling Different Types of Angles

### Positive Angles > 360°

Subtract 360° repeatedly until the angle is in [0°, 360°).

**Example:** 840°
$$840° - 360° = 480°$$
$$480° - 360° = 120°$$

840° is coterminal with 120°.

### Negative Angles

Add 360° repeatedly until the angle is in [0°, 360°).

**Example:** -135°
$$-135° + 360° = 225°$$

-135° is coterminal with 225°.

### Very Large Angles

Use modular arithmetic: θ (mod 360°)

**Example:** 1290°
$$1290° = 3 \times 360° + 210°$$
$$1290° \equiv 210° \pmod{360°}$$

---

## 📊 Complete Reference Tables

### First Quadrant (Standard Values)

| θ | sin θ | cos θ | tan θ |
|---|-------|-------|-------|
| 0° | 0 | 1 | 0 |
| 30° | 1/2 | √3/2 | √3/3 |
| 45° | √2/2 | √2/2 | 1 |
| 60° | √3/2 | 1/2 | √3 |
| 90° | 1 | 0 | undefined |

### Second Quadrant

| θ | Ref Angle | sin θ | cos θ | tan θ |
|---|-----------|-------|-------|-------|
| 120° | 60° | +√3/2 | -1/2 | -√3 |
| 135° | 45° | +√2/2 | -√2/2 | -1 |
| 150° | 30° | +1/2 | -√3/2 | -√3/3 |
| 180° | 0° | 0 | -1 | 0 |

### Third Quadrant

| θ | Ref Angle | sin θ | cos θ | tan θ |
|---|-----------|-------|-------|-------|
| 210° | 30° | -1/2 | -√3/2 | +√3/3 |
| 225° | 45° | -√2/2 | -√2/2 | +1 |
| 240° | 60° | -√3/2 | -1/2 | +√3 |
| 270° | 90° | -1 | 0 | undefined |

### Fourth Quadrant

| θ | Ref Angle | sin θ | cos θ | tan θ |
|---|-----------|-------|-------|-------|
| 300° | 60° | -√3/2 | +1/2 | -√3 |
| 315° | 45° | -√2/2 | +√2/2 | -1 |
| 330° | 30° | -1/2 | +√3/2 | -√3/3 |
| 360° | 0° | 0 | 1 | 0 |

---

## 🎯 Complete Unit Circle with All Values

```
                                    90° (π/2)
                                     (0, 1)
                                       │
                                       │
         120° (2π/3)                   │                   60° (π/3)
        (-1/2, √3/2)                   │                  (1/2, √3/2)
               \                       │                       /
                \                      │                      /
     135° (3π/4) \                     │                     / 45° (π/4)
    (-√2/2, √2/2) \                    │                    / (√2/2, √2/2)
                   \                   │                   /
                    \                  │                  /
      150° (5π/6)    \                 │                 /    30° (π/6)
     (-√3/2, 1/2)     \                │                /    (√3/2, 1/2)
                       \               │               /
                        \              │              /
                         \             │             /
    180° (π) ─────────────\────────────●────────────/─────────── 0° (0)
    (-1, 0)                \           │           /              (1, 0)
                            \          │          /
                             \         │         /
      210° (7π/6)             \        │        /              330° (11π/6)
     (-√3/2, -1/2)             \       │       /              (√3/2, -1/2)
                                \      │      /
                                 \     │     /
     225° (5π/4)                  \    │    /                315° (7π/4)
    (-√2/2, -√2/2)                 \   │   /                (√2/2, -√2/2)
                                    \  │  /
         240° (4π/3)                 \ │ /                 300° (5π/3)
        (-1/2, -√3/2)                 \│/                 (1/2, -√3/2)
                                       │
                                       │
                                    (0, -1)
                                   270° (3π/2)
```

---

## 🧮 Worked Examples

### Example 1: Positive Large Angle

Find sin 570°.

**Solution:**

Step 1: Normalize
$$570° - 360° = 210°$$

Step 2: Find reference angle
$$210° \text{ is in Q III}$$
$$\alpha = 210° - 180° = 30°$$

Step 3: Determine sign and calculate
- In Q III, sin is negative
- sin 570° = sin 210° = -sin 30° = **-1/2**

### Example 2: Negative Angle

Find cos(-240°).

**Solution:**

Step 1: Normalize
$$-240° + 360° = 120°$$

Step 2: Find reference angle
$$120° \text{ is in Q II}$$
$$\alpha = 180° - 120° = 60°$$

Step 3: Determine sign and calculate
- In Q II, cos is negative
- cos(-240°) = cos 120° = -cos 60° = **-1/2**

### Example 3: Tangent with Large Angle

Find tan 495°.

**Solution:**

Step 1: Normalize
$$495° - 360° = 135°$$

Step 2: Find reference angle
$$135° \text{ is in Q II}$$
$$\alpha = 180° - 135° = 45°$$

Step 3: Determine sign and calculate
- In Q II, tan is negative
- tan 495° = tan 135° = -tan 45° = **-1**

### Example 4: Multiple Rotations

Find sin 1110°.

**Solution:**

Step 1: Normalize
$$1110° = 3 \times 360° + 30°$$
$$1110° - 1080° = 30°$$

Step 2: Reference angle = 30° (Q I)

Step 3: In Q I, sin is positive
- sin 1110° = sin 30° = **1/2**

### Example 5: Negative Angle in Radians

Find cos(-5π/6).

**Solution:**

Step 1: Normalize
$$-5\pi/6 + 2\pi = -5\pi/6 + 12\pi/6 = 7\pi/6$$

Step 2: Find reference angle
$$7\pi/6 \text{ is in Q III}$$
$$\alpha = 7\pi/6 - \pi = 7\pi/6 - 6\pi/6 = \pi/6$$

Step 3: Determine sign and calculate
- In Q III, cos is negative
- cos(-5π/6) = cos(7π/6) = -cos(π/6) = **-√3/2**

### Example 6: All Six Functions

Find all six trigonometric ratios for θ = 225°.

**Solution:**

Step 1: 225° is in Q III

Step 2: Reference angle = 225° - 180° = 45°

Step 3: Signs in Q III: sin(-), cos(-), tan(+)

| Function | Calculation | Value |
|----------|-------------|-------|
| sin 225° | -sin 45° | **-√2/2** |
| cos 225° | -cos 45° | **-√2/2** |
| tan 225° | +tan 45° | **1** |
| csc 225° | -csc 45° | **-√2** |
| sec 225° | -sec 45° | **-√2** |
| cot 225° | +cot 45° | **1** |

---

## 📐 Special Cases

### Quadrantal Angles

These are angles on the axes (0°, 90°, 180°, 270°, 360°).

```
    Quadrantal Angle Values:
    
    ┌───────┬───────┬───────┬───────┬─────────────┐
    │   θ   │ sin θ │ cos θ │ tan θ │   Point     │
    ├───────┼───────┼───────┼───────┼─────────────┤
    │   0°  │   0   │   1   │   0   │   (1, 0)    │
    │  90°  │   1   │   0   │  undef│   (0, 1)    │
    │ 180°  │   0   │  -1   │   0   │  (-1, 0)    │
    │ 270°  │  -1   │   0   │  undef│   (0, -1)   │
    │ 360°  │   0   │   1   │   0   │   (1, 0)    │
    └───────┴───────┴───────┴───────┴─────────────┘
```

### Odd Multiples of 90°

| Angle | Equals | sin | cos |
|-------|--------|-----|-----|
| 90° | 90° | 1 | 0 |
| 270° | -90° | -1 | 0 |
| 450° | 90° | 1 | 0 |
| -90° | 270° | -1 | 0 |

---

## 🔢 Periodic Properties

### Period of Trigonometric Functions

| Function | Period |
|----------|--------|
| sin θ | 360° (2π) |
| cos θ | 360° (2π) |
| tan θ | 180° (π) |
| cot θ | 180° (π) |
| sec θ | 360° (2π) |
| csc θ | 360° (2π) |

### Using Periodicity

For any integer n:
$$\sin(\theta + 360°n) = \sin \theta$$
$$\cos(\theta + 360°n) = \cos \theta$$
$$\tan(\theta + 180°n) = \tan \theta$$

---

## 📈 Visualization of Extended Angles

### Sine Wave for Multiple Rotations

```
    sin θ
      1 │      ●           ●           ●
        │    /   \       /   \       /   \
        │   /     \     /     \     /     \
      0 ├──●───────●───●───────●───●───────●── θ
        │           \ /         \ /         \
        │            ●           ●           ●
     -1 │
        └──────────────────────────────────────
           0°  180° 360° 540° 720° 900° 1080°
           
    Pattern repeats every 360°
```

### Tangent with Asymptotes

```
    tan θ
        │         │         │         │
        │    /    │    /    │    /    │
        │   /     │   /     │   /     │
      0 ├─/───────┼─/───────┼─/───────┼── θ
        /         │/        │/        │
       /│         /│        /│        │
        │         │         │         │
        └─────────┴─────────┴─────────┴──
        0°   90° 180° 270° 360° 450° 540°
        
    Asymptotes at 90°, 270°, 450°...
    Pattern repeats every 180°
```

---

## 🔗 Negative Angle Identities

### Odd and Even Functions

| Function | Type | Identity |
|----------|------|----------|
| sin(-θ) | Odd | -sin θ |
| cos(-θ) | Even | cos θ |
| tan(-θ) | Odd | -tan θ |
| cot(-θ) | Odd | -cot θ |
| sec(-θ) | Even | sec θ |
| csc(-θ) | Odd | -csc θ |

### Using These Identities

Instead of normalizing negative angles, you can use:

**Example:** sin(-30°) = -sin 30° = -1/2

**Example:** cos(-60°) = cos 60° = 1/2

---

## 🌍 Real-World Applications

### 1. Rotating Machinery
Angles beyond 360° represent multiple rotations of gears, wheels, and motors.

### 2. Signal Processing
Negative angles appear in phase-shifted signals and time-delayed waves.

### 3. Astronomy
Planetary positions use angles that extend over many rotations.

### 4. Navigation
Aircraft and ship headings can accumulate through multiple rotations during maneuvers.

---

## 📋 Summary Table

### Master Process

| Step | Action | Example (θ = 510°) |
|------|--------|-------------------|
| 1 | Normalize | 510° - 360° = 150° |
| 2 | Find quadrant | Q II |
| 3 | Reference angle | 180° - 150° = 30° |
| 4 | ASTC sign | sin + in Q II |
| 5 | Calculate | sin 510° = +sin 30° = 1/2 |

### Quick Reference

| Angle Type | First Step |
|------------|------------|
| θ > 360° | Subtract 360° |
| θ < 0° | Add 360° |
| θ in radians > 2π | Subtract 2π |
| θ in radians < 0 | Add 2π |

### Function Properties

| Property | sin | cos | tan |
|----------|-----|-----|-----|
| Period | 360° | 360° | 180° |
| f(-θ) | -sin θ | cos θ | -tan θ |
| Range | [-1, 1] | [-1, 1] | (-∞, ∞) |

---

## ❓ Quick Revision Questions

1. **Find sin 405°.**

2. **Find cos(-315°).**

3. **Find tan 600°.**

4. **Find all six trigonometric ratios for θ = -150°.**

5. **If sin θ = 1/2 and 0° ≤ θ < 360°, find all possible values of θ.**

6. **Explain why tan 450° = tan 90° = undefined.**

<details>
<summary>Click to see answers</summary>

1. 405° - 360° = 45° (Q I)  
   sin 405° = sin 45° = **√2/2**

2. -315° + 360° = 45° (Q I)  
   cos(-315°) = cos 45° = **√2/2**  
   Or: cos(-315°) = cos(315°) = √2/2 (cosine is even)

3. 600° - 360° = 240° (Q III)  
   Reference angle = 240° - 180° = 60°  
   In Q III, tan is positive  
   tan 600° = tan 60° = **√3**

4. -150° + 360° = 210° (Q III)  
   Reference angle = 30°  
   sin(-150°) = -1/2, cos(-150°) = -√3/2  
   tan(-150°) = 1/√3 = √3/3  
   csc(-150°) = -2, sec(-150°) = -2√3/3  
   cot(-150°) = √3

5. sin θ = 1/2 when θ = **30°** (Q I) or θ = **150°** (Q II)

6. 450° - 360° = 90°  
   At 90°, the point is (0, 1)  
   tan 90° = y/x = 1/0 = **undefined** (division by zero)

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 2.3 Reference Angles](03-reference-angles.md) | [Unit 2 Index](README.md) | [Unit 3: Trigonometric Identities →](../03-Trigonometric-Identities/README.md) |

---

[← Back to Main Index](../README.md)
