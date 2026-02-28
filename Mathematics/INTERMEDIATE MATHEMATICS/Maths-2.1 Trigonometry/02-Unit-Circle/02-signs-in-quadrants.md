# Chapter 2.2: Signs in Different Quadrants (ASTC Rule)

## Overview

When working with angles beyond the first quadrant (0° to 90°), trigonometric functions can be positive or negative depending on the quadrant. This chapter explains how to determine the sign of any trigonometric function for any angle using the **ASTC rule**.

---

## 📐 The Four Quadrants

The coordinate plane is divided into four quadrants by the x and y axes:

```
                           y
                           │
                           │
         Quadrant II       │       Quadrant I
                           │
      x < 0, y > 0         │      x > 0, y > 0
                           │
    ───────────────────────●─────────────────────── x
                           │
      x < 0, y < 0         │      x > 0, y < 0
                           │
        Quadrant III       │      Quadrant IV
                           │
                           │
```

### Quadrant Boundaries

| Quadrant | Angle Range (Degrees) | Angle Range (Radians) |
|----------|----------------------|----------------------|
| I | 0° < θ < 90° | 0 < θ < π/2 |
| II | 90° < θ < 180° | π/2 < θ < π |
| III | 180° < θ < 270° | π < θ < 3π/2 |
| IV | 270° < θ < 360° | 3π/2 < θ < 2π |

---

## 🔍 Signs Based on Coordinates

Since on the unit circle:
- **sin θ = y-coordinate**
- **cos θ = x-coordinate**
- **tan θ = y/x**

The signs depend on the signs of x and y in each quadrant:

```
                           y
                           │
                           │
         x < 0             │             x > 0
         y > 0             │             y > 0
                           │
         sin θ = + ────────┼──────── sin θ = +
         cos θ = -         │         cos θ = +
         tan θ = -         │         tan θ = +
                           │
    ───────────────────────●─────────────────────── x
                           │
         x < 0             │             x > 0
         y < 0             │             y < 0
                           │
         sin θ = - ────────┼──────── sin θ = -
         cos θ = -         │         cos θ = +
         tan θ = +         │         tan θ = -
                           │
```

---

## ⭐ The ASTC Rule

### The Mnemonic

**A**ll **S**tudents **T**ake **C**alculus

or

**A**dd **S**ugar **T**o **C**offee

```
                           │
                           │
            S              │            A
         (Sine +)          │         (All +)
                           │
    ───────────────────────●─────────────────────── 
                           │
            T              │            C
        (Tangent +)        │        (Cosine +)
                           │
```

### What ASTC Means

| Quadrant | Letter | Meaning | Positive Functions |
|----------|--------|---------|-------------------|
| I | **A** | **A**ll | sin, cos, tan (all positive) |
| II | **S** | **S**ine | Only sin (and csc) positive |
| III | **T** | **T**angent | Only tan (and cot) positive |
| IV | **C** | **C**osine | Only cos (and sec) positive |

---

## 📊 Complete Sign Chart

### Primary Functions

| Function | Quadrant I | Quadrant II | Quadrant III | Quadrant IV |
|----------|-----------|------------|-------------|------------|
| sin θ | + | + | - | - |
| cos θ | + | - | - | + |
| tan θ | + | - | + | - |

### Reciprocal Functions

| Function | Quadrant I | Quadrant II | Quadrant III | Quadrant IV |
|----------|-----------|------------|-------------|------------|
| csc θ | + | + | - | - |
| sec θ | + | - | - | + |
| cot θ | + | - | + | - |

### Visual Representation

```
    ┌─────────────────────────────────────────────────────────┐
    │                    SIGN CHART                           │
    ├─────────────────────────────────────────────────────────┤
    │                         │                               │
    │      Quadrant II        │        Quadrant I             │
    │                         │                               │
    │   sin θ = +             │    sin θ = +                  │
    │   cos θ = -             │    cos θ = +                  │
    │   tan θ = -             │    tan θ = +                  │
    │   ──────────────────────┼──────────────────────         │
    │   csc θ = +             │    csc θ = +                  │
    │   sec θ = -             │    sec θ = +                  │
    │   cot θ = -             │    cot θ = +                  │
    │                         │                               │
    ├─────────────────────────┼───────────────────────────────┤
    │                         │                               │
    │      Quadrant III       │        Quadrant IV            │
    │                         │                               │
    │   sin θ = -             │    sin θ = -                  │
    │   cos θ = -             │    cos θ = +                  │
    │   tan θ = +             │    tan θ = -                  │
    │   ──────────────────────┼──────────────────────         │
    │   csc θ = -             │    csc θ = -                  │
    │   sec θ = +             │    sec θ = +                  │
    │   cot θ = +             │    cot θ = -                  │
    │                         │                               │
    └─────────────────────────────────────────────────────────┘
```

---

## 🧠 Why Does This Work?

### Quadrant I (0° < θ < 90°)
- Point P is in upper right: x > 0, y > 0
- sin θ = y > 0 ✓
- cos θ = x > 0 ✓
- tan θ = y/x > 0 ✓

### Quadrant II (90° < θ < 180°)
- Point P is in upper left: x < 0, y > 0
- sin θ = y > 0 ✓
- cos θ = x < 0 ✓
- tan θ = y/x < 0 ✓ (positive ÷ negative)

### Quadrant III (180° < θ < 270°)
- Point P is in lower left: x < 0, y < 0
- sin θ = y < 0 ✓
- cos θ = x < 0 ✓
- tan θ = y/x > 0 ✓ (negative ÷ negative)

### Quadrant IV (270° < θ < 360°)
- Point P is in lower right: x > 0, y < 0
- sin θ = y < 0 ✓
- cos θ = x > 0 ✓
- tan θ = y/x < 0 ✓ (negative ÷ positive)

---

## 📐 Signs at Quadrantal Angles

At angles on the axes (0°, 90°, 180°, 270°), some functions equal zero or are undefined:

| Angle | sin | cos | tan |
|-------|-----|-----|-----|
| 0° | 0 | 1 | 0 |
| 90° | 1 | 0 | undefined |
| 180° | 0 | -1 | 0 |
| 270° | -1 | 0 | undefined |
| 360° | 0 | 1 | 0 |

```
    At Quadrantal Angles:
    
                    90°
                  sin = 1
                  cos = 0
                  tan = ∞
                     │
                     │
    180° ────────────●──────────── 0°/360°
    sin = 0          │            sin = 0
    cos = -1         │            cos = 1
    tan = 0          │            tan = 0
                     │
                     │
                   270°
                  sin = -1
                  cos = 0
                  tan = ∞
```

---

## 🧮 Worked Examples

### Example 1: Determine the Quadrant

If sin θ > 0 and cos θ < 0, in which quadrant does θ lie?

**Solution:**
- sin θ > 0: Quadrants I or II
- cos θ < 0: Quadrants II or III
- Both conditions: **Quadrant II**

### Example 2: Find the Sign

Determine the sign of tan 235°.

**Solution:**
235° is between 180° and 270° → **Quadrant III**

In Quadrant III, tan is positive (T in ASTC)

**tan 235° is positive**

### Example 3: Find All Possible Quadrants

If sin θ = -3/5, in which quadrant(s) could θ lie?

**Solution:**
sin θ < 0 means y-coordinate is negative

This occurs in **Quadrant III or Quadrant IV**

### Example 4: Complete Set of Ratios

Given cos θ = -4/5 and θ is in Quadrant II, find all six trigonometric ratios.

**Solution:**

Using the Pythagorean identity:
$$\sin^2 \theta = 1 - \cos^2 \theta = 1 - \frac{16}{25} = \frac{9}{25}$$
$$\sin \theta = \pm \frac{3}{5}$$

In Quadrant II, sin is positive: **sin θ = 3/5**

| Ratio | Calculation | Value |
|-------|-------------|-------|
| sin θ | (positive in Q II) | 3/5 |
| cos θ | (given) | -4/5 |
| tan θ | sin/cos = (3/5)/(-4/5) | -3/4 |
| cot θ | 1/tan θ | -4/3 |
| sec θ | 1/cos θ | -5/4 |
| csc θ | 1/sin θ | 5/3 |

### Example 5: Identifying Quadrant from Values

If tan θ = 2 and sin θ < 0, find the quadrant and cos θ.

**Solution:**

- tan θ > 0: Quadrants I or III
- sin θ < 0: Quadrants III or IV
- Both conditions: **Quadrant III**

In Q III, both sin and cos are negative.

Using tan θ = sin θ/cos θ = 2, let sin θ = -2k, cos θ = -k

From sin²θ + cos²θ = 1:
$$4k^2 + k^2 = 1$$
$$5k^2 = 1$$
$$k = \frac{1}{\sqrt{5}}$$

**cos θ = -1/√5 = -√5/5**

---

## 📈 Sign Pattern Visualization

### Sine Function Signs

```
           │ + + + +
           │ + + +
           │ + +
           │ +
    ───────●───────
           │ -
           │ - -
           │ - - -
           │ - - - -
           
    Positive: 0° to 180° (top half)
    Negative: 180° to 360° (bottom half)
```

### Cosine Function Signs

```
     - - - │ + + +
     - - - │ + + +
     - - - │ + + +
     - - - │ + + +
    ───────●───────
     - - - │ + + +
     - - - │ + + +
     - - - │ + + +
     - - - │ + + +
    
    Positive: -90° to 90° (right half)
    Negative: 90° to 270° (left half)
```

### Tangent Function Signs

```
       -   │   +
        -  │  +
         - │ +
           │
    ───────●───────
           │
         + │ -
        +  │  -
       +   │   -
       
    Positive: Quadrants I and III (diagonal)
    Negative: Quadrants II and IV (diagonal)
```

---

## 🔄 Alternative Mnemonics

### "All Stations To Central"
- **A**ll (Quadrant I)
- **S**tations (Quadrant II - Sine)
- **T**o (Quadrant III - Tangent)
- **C**entral (Quadrant IV - Cosine)

### "CAST" (reading counter-clockwise from Q IV)
Start from Quadrant IV and read counter-clockwise:
**C**-**A**-**S**-**T**

### Visual Memory Aid

```
        ┌───────────────────────────────┐
        │                               │
        │   Think of the unit circle:   │
        │                               │
        │      S │ A     ← All positive │
        │     ───┼───       in Q I      │
        │      T │ C                    │
        │                               │
        │   Reading: Start at Q I (A)   │
        │   Go counter-clockwise:       │
        │   A → S → T → C               │
        │                               │
        └───────────────────────────────┘
```

---

## 🌍 Real-World Applications

### 1. Physics - Projectile Motion
Components of velocity in different directions require understanding signs based on direction.

### 2. Engineering - Force Analysis
Forces acting in different quadrants have positive or negative components.

### 3. Navigation
Bearings and directions use angle measurements that span all four quadrants.

### 4. Signal Processing
Alternating current (AC) signals have positive and negative phases following sine wave patterns.

---

## 📋 Summary Table

| Quadrant | Angle Range | Positive Functions | Memory |
|----------|-------------|-------------------|--------|
| I | 0° - 90° | All (sin, cos, tan) | **A**ll |
| II | 90° - 180° | Sine (and csc) | **S**tudents |
| III | 180° - 270° | Tangent (and cot) | **T**ake |
| IV | 270° - 360° | Cosine (and sec) | **C**alculus |

### Quick Decision Tree

```
    Is sin θ positive?
    ├── Yes → Quadrant I or II
    │         Is cos θ positive?
    │         ├── Yes → Quadrant I
    │         └── No  → Quadrant II
    │
    └── No  → Quadrant III or IV
              Is cos θ positive?
              ├── Yes → Quadrant IV
              └── No  → Quadrant III
```

---

## ❓ Quick Revision Questions

1. **In which quadrant(s) is tan θ negative?**

2. **If cos θ = 0.5 and sin θ < 0, in which quadrant is θ?**

3. **What is the sign of sin 315°?**

4. **If sec θ < 0 and cot θ > 0, find the quadrant.**

5. **Complete: In Quadrant III, _____ and _____ are the only positive functions.**

6. **Given sin θ = -5/13 and cos θ = 12/13, find tan θ and the quadrant.**

<details>
<summary>Click to see answers</summary>

1. **Quadrants II and IV** (where sine and cosine have opposite signs)

2. cos θ > 0: Quadrants I or IV  
   sin θ < 0: Quadrants III or IV  
   Both: **Quadrant IV**

3. 315° is in Quadrant IV (between 270° and 360°)  
   sin is negative in Q IV  
   **sin 315° is negative**

4. sec θ < 0 means cos θ < 0: Quadrants II or III  
   cot θ > 0 means tan θ > 0: Quadrants I or III  
   Both: **Quadrant III**

5. **Tangent** and **Cotangent**

6. tan θ = sin θ/cos θ = (-5/13)/(12/13) = **-5/12**  
   sin θ < 0 and cos θ > 0 → **Quadrant IV**

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 2.1 Unit Circle Definition](01-unit-circle-definition.md) | [Unit 2 Index](README.md) | [2.3 Reference Angles →](03-reference-angles.md) |

---

[← Back to Main Index](../README.md)
