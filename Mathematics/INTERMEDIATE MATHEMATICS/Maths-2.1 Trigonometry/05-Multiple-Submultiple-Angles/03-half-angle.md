# Chapter 5.3: Half Angle Formulas

## Overview

Half angle formulas express trigonometric functions of **A/2** in terms of functions of **A**. They are derived from the double angle formulas by replacing A with A/2. These formulas are particularly useful for finding exact values and in integration.

---

## 📐 The Half Angle Formulas

### Main Formulas

$$\boxed{\sin\frac{A}{2} = \pm\sqrt{\frac{1 - \cos A}{2}}}$$

$$\boxed{\cos\frac{A}{2} = \pm\sqrt{\frac{1 + \cos A}{2}}}$$

$$\boxed{\tan\frac{A}{2} = \pm\sqrt{\frac{1 - \cos A}{1 + \cos A}}}$$

### Alternative Forms for tan(A/2) (No ± ambiguity)

$$\boxed{\tan\frac{A}{2} = \frac{\sin A}{1 + \cos A}}$$

$$\boxed{\tan\frac{A}{2} = \frac{1 - \cos A}{\sin A}}$$

---

## 🔍 Derivations

### Deriving sin(A/2)

Start with the double angle formula:
$$\cos A = 1 - 2\sin^2\frac{A}{2}$$

Rearranging:
$$2\sin^2\frac{A}{2} = 1 - \cos A$$
$$\sin^2\frac{A}{2} = \frac{1 - \cos A}{2}$$
$$\sin\frac{A}{2} = \pm\sqrt{\frac{1 - \cos A}{2}}$$

### Deriving cos(A/2)

Start with:
$$\cos A = 2\cos^2\frac{A}{2} - 1$$

Rearranging:
$$2\cos^2\frac{A}{2} = 1 + \cos A$$
$$\cos^2\frac{A}{2} = \frac{1 + \cos A}{2}$$
$$\cos\frac{A}{2} = \pm\sqrt{\frac{1 + \cos A}{2}}$$

### Deriving tan(A/2) - Main Form

$$\tan\frac{A}{2} = \frac{\sin\frac{A}{2}}{\cos\frac{A}{2}}$$
$$= \frac{\pm\sqrt{\frac{1 - \cos A}{2}}}{\pm\sqrt{\frac{1 + \cos A}{2}}}$$
$$= \pm\sqrt{\frac{1 - \cos A}{1 + \cos A}}$$

### Deriving Alternative Forms for tan(A/2)

**Form 1:** Multiply by sin A / sin A

Starting from: $\tan\frac{A}{2} = \frac{1 - \cos A}{\sin A}$

$$\tan\frac{A}{2} = \frac{\sin\frac{A}{2}}{\cos\frac{A}{2}} \cdot \frac{2\cos\frac{A}{2}}{2\cos\frac{A}{2}}$$
$$= \frac{2\sin\frac{A}{2}\cos\frac{A}{2}}{2\cos^2\frac{A}{2}}$$
$$= \frac{\sin A}{1 + \cos A}$$

**Form 2:** Similarly:
$$\tan\frac{A}{2} = \frac{2\sin^2\frac{A}{2}}{2\sin\frac{A}{2}\cos\frac{A}{2}}$$
$$= \frac{1 - \cos A}{\sin A}$$

---

## 📊 Formula Reference Chart

```
    ┌─────────────────────────────────────────────────────────────────┐
    │                     HALF ANGLE FORMULAS                         │
    ├─────────────────────────────────────────────────────────────────┤
    │                                                                 │
    │  sin(A/2) = ± √[(1 - cos A)/2]                                 │
    │                                                                 │
    │  cos(A/2) = ± √[(1 + cos A)/2]                                 │
    │                                                                 │
    │  tan(A/2) = ± √[(1 - cos A)/(1 + cos A)]                       │
    │                                                                 │
    │  ┌───────────────────────────────────────────────────────────┐ │
    │  │     ALTERNATIVE tan(A/2) FORMS (NO ± SIGN!)              │ │
    │  ├───────────────────────────────────────────────────────────┤ │
    │  │                                                           │ │
    │  │  tan(A/2) = sin A / (1 + cos A)                          │ │
    │  │                                                           │ │
    │  │  tan(A/2) = (1 - cos A) / sin A                          │ │
    │  │                                                           │ │
    │  └───────────────────────────────────────────────────────────┘ │
    │                                                                 │
    └─────────────────────────────────────────────────────────────────┘
```

---

## 🧠 Determining the Sign (±)

### Quadrant Analysis

The sign of sin(A/2), cos(A/2), and tan(A/2) depends on which quadrant A/2 falls in.

```
    Given angle A, find which quadrant A/2 is in:
    
    If A is in:           Then A/2 is in:
    ───────────           ─────────────────
    0° to 180°      →     0° to 90° (Q1)
    180° to 360°    →     90° to 180° (Q2)
    360° to 540°    →     180° to 270° (Q3)
    540° to 720°    →     270° to 360° (Q4)
```

### Sign Chart for A/2

| A/2 in Quadrant | sin(A/2) | cos(A/2) | tan(A/2) |
|-----------------|----------|----------|----------|
| Q1 (0° - 90°) | + | + | + |
| Q2 (90° - 180°) | + | - | - |
| Q3 (180° - 270°) | - | - | + |
| Q4 (270° - 360°) | - | + | - |

---

## 📐 Visualization

### Half Angle on Unit Circle

```
    The angle A/2 is exactly half of angle A
    
              y
              │      A/2
              │     ╱
              │   ╱ half the arc
              │ ╱
              │╱ A (full angle)
    ──────────*────────────── x
              │╲
              │  ╲
              │    ╲
              │      ╲
              
    If A = 120°, then A/2 = 60°
    If A = 90°, then A/2 = 45°
```

---

## 🧮 Worked Examples

### Example 1: Find sin(22.5°)

**Solution:**
22.5° = 45°/2, so we use A = 45°.

Since 22.5° is in Q1, sin(22.5°) is positive.

$$\sin 22.5° = \sqrt{\frac{1 - \cos 45°}{2}}$$
$$= \sqrt{\frac{1 - \frac{\sqrt{2}}{2}}{2}}$$
$$= \sqrt{\frac{2 - \sqrt{2}}{4}}$$
$$= \boxed{\frac{\sqrt{2 - \sqrt{2}}}{2}}$$

### Example 2: Find cos(15°)

**Solution:**
15° = 30°/2, so we use A = 30°.

Since 15° is in Q1, cos(15°) is positive.

$$\cos 15° = \sqrt{\frac{1 + \cos 30°}{2}}$$
$$= \sqrt{\frac{1 + \frac{\sqrt{3}}{2}}{2}}$$
$$= \sqrt{\frac{2 + \sqrt{3}}{4}}$$
$$= \boxed{\frac{\sqrt{2 + \sqrt{3}}}{2}}$$

Note: This can also be written as $\frac{\sqrt{6} + \sqrt{2}}{4}$ (using compound angle formula).

### Example 3: Find tan(75°/2) = tan(37.5°)

**Solution:**
Using tan(A/2) = sin A/(1 + cos A) with A = 75°:

First, find sin 75° and cos 75°:
- sin 75° = sin(45° + 30°) = $\frac{\sqrt{6} + \sqrt{2}}{4}$
- cos 75° = cos(45° + 30°) = $\frac{\sqrt{6} - \sqrt{2}}{4}$

$$\tan 37.5° = \frac{\frac{\sqrt{6} + \sqrt{2}}{4}}{1 + \frac{\sqrt{6} - \sqrt{2}}{4}}$$
$$= \frac{\sqrt{6} + \sqrt{2}}{4 + \sqrt{6} - \sqrt{2}}$$

This can be simplified but the exact form is complex.

### Example 4: Using Given Information

If cos A = 3/5 and 270° < A < 360°, find sin(A/2), cos(A/2), and tan(A/2).

**Solution:**
First, determine the quadrant for A/2:
- A is in Q4 (270° to 360°)
- So A/2 is in 135° to 180° (Q2)

In Q2: sin is +, cos is -, tan is -

$$\sin\frac{A}{2} = +\sqrt{\frac{1 - \frac{3}{5}}{2}} = \sqrt{\frac{\frac{2}{5}}{2}} = \sqrt{\frac{1}{5}} = \boxed{\frac{1}{\sqrt{5}}}$$

$$\cos\frac{A}{2} = -\sqrt{\frac{1 + \frac{3}{5}}{2}} = -\sqrt{\frac{\frac{8}{5}}{2}} = -\sqrt{\frac{4}{5}} = \boxed{-\frac{2}{\sqrt{5}}}$$

$$\tan\frac{A}{2} = \frac{\sin\frac{A}{2}}{\cos\frac{A}{2}} = \frac{\frac{1}{\sqrt{5}}}{-\frac{2}{\sqrt{5}}} = \boxed{-\frac{1}{2}}$$

### Example 5: Proving an Identity

Prove that $\tan\frac{A}{2} = \csc A - \cot A$

**Solution:**
$$\text{RHS} = \csc A - \cot A$$
$$= \frac{1}{\sin A} - \frac{\cos A}{\sin A}$$
$$= \frac{1 - \cos A}{\sin A}$$
$$= \tan\frac{A}{2} = \text{LHS} \quad \checkmark$$

### Example 6: Express in Terms of Half Angle

Express (1 - cos A)/sin A in terms of A/2.

**Solution:**
$$\frac{1 - \cos A}{\sin A} = \frac{2\sin^2\frac{A}{2}}{2\sin\frac{A}{2}\cos\frac{A}{2}}$$
$$= \frac{\sin\frac{A}{2}}{\cos\frac{A}{2}}$$
$$= \boxed{\tan\frac{A}{2}}$$

---

## 📊 Important Relations

### Summary of tan(A/2) Forms

```
    ┌─────────────────────────────────────────────────────────────────┐
    │                  ALL FORMS OF tan(A/2)                          │
    ├─────────────────────────────────────────────────────────────────┤
    │                                                                 │
    │  tan(A/2) = ± √[(1 - cos A)/(1 + cos A)]   (ambiguous sign)    │
    │                                                                 │
    │  tan(A/2) = sin A/(1 + cos A)              (no ambiguity)       │
    │                                                                 │
    │  tan(A/2) = (1 - cos A)/sin A              (no ambiguity)       │
    │                                                                 │
    │  tan(A/2) = csc A - cot A                  (identity form)      │
    │                                                                 │
    └─────────────────────────────────────────────────────────────────┘
```

### Weierstrass Substitution

Let $t = \tan\frac{A}{2}$. Then:

$$\sin A = \frac{2t}{1 + t^2}$$

$$\cos A = \frac{1 - t^2}{1 + t^2}$$

$$\tan A = \frac{2t}{1 - t^2}$$

This is extremely useful in integration!

---

## 🌍 Applications

### 1. Weierstrass Substitution in Calculus

To integrate $\int \frac{1}{a + b\sin x + c\cos x} dx$:

Let $t = \tan(x/2)$, then use:
$$\sin x = \frac{2t}{1+t^2}, \quad \cos x = \frac{1-t^2}{1+t^2}, \quad dx = \frac{2}{1+t^2}dt$$

### 2. Finding Exact Values

For angles like 22.5°, 67.5°, 112.5°, etc.

### 3. Solving Trigonometric Equations

Equations involving both sin and cos can often be simplified.

### 4. Computer Graphics

Rotation by half-angles is used in quaternion-based rotations.

---

## 📋 Summary Table

### Formula Quick Reference

| Formula | Expression |
|---------|------------|
| sin(A/2) | ±√[(1 - cos A)/2] |
| cos(A/2) | ±√[(1 + cos A)/2] |
| tan(A/2) | ±√[(1 - cos A)/(1 + cos A)] |
| tan(A/2) | sin A/(1 + cos A) |
| tan(A/2) | (1 - cos A)/sin A |

### Weierstrass Substitution (t = tan(A/2))

| Expression | In Terms of t |
|------------|---------------|
| sin A | 2t/(1 + t²) |
| cos A | (1 - t²)/(1 + t²) |
| tan A | 2t/(1 - t²) |

### Key Points

| Concept | Detail |
|---------|--------|
| Sign choice | Depends on quadrant of A/2 |
| Alternative tan forms | Have no ± ambiguity |
| Memory aid for sin | 1 MINUS cos in numerator |
| Memory aid for cos | 1 PLUS cos in numerator |
| Weierstrass | Converts all trig to rational functions |

---

## ❓ Quick Revision Questions

1. **Find sin(π/8) using the half-angle formula. (Note: π/8 = 22.5°)**

2. **If cos θ = 4/5 and θ is in Q4, find tan(θ/2).**

3. **Prove that (1 + cos A)/sin A = cot(A/2)**

4. **Express sin A in terms of t = tan(A/2).**

5. **Find the exact value of cos(67.5°). (Hint: 67.5° = 135°/2)**

6. **Why do the alternative forms of tan(A/2) not require a ± sign?**

<details>
<summary>Click to see answers</summary>

1. sin(π/8) = sin(22.5°) = sin(45°/2)  
   = √[(1 - cos 45°)/2]  
   = √[(1 - √2/2)/2]  
   = √[(2 - √2)/4]  
   = **√(2 - √2)/2**

2. θ in Q4 means 270° < θ < 360°  
   So 135° < θ/2 < 180° (Q2)  
   In Q2, tan is negative  
   
   tan(θ/2) = (1 - cos θ)/sin θ  
   Need sin θ: sin θ = -√(1 - 16/25) = -3/5 (negative in Q4)  
   
   tan(θ/2) = (1 - 4/5)/(-3/5)  
   = (1/5)/(-3/5)  
   = **-1/3**

3. LHS = (1 + cos A)/sin A  
   = (2cos²(A/2))/(2sin(A/2)cos(A/2))  
   = cos(A/2)/sin(A/2)  
   = **cot(A/2)** ✓

4. Let t = tan(A/2)  
   sin A = 2sin(A/2)cos(A/2)  
   = 2 · (t/sec(A/2)) · (1/sec(A/2))  
   = 2t/sec²(A/2)  
   = 2t/(1 + tan²(A/2))  
   = **2t/(1 + t²)**

5. cos(67.5°) = cos(135°/2)  
   Since 67.5° is in Q1, cos is positive  
   = √[(1 + cos 135°)/2]  
   = √[(1 - √2/2)/2]  
   = √[(2 - √2)/4]  
   = **√(2 - √2)/2**

6. The forms tan(A/2) = sin A/(1 + cos A) and tan(A/2) = (1 - cos A)/sin A are derived using double angle formulas without taking square roots. Since sin A and cos A already carry their proper signs based on the quadrant of A, these formulas automatically give the correct sign for tan(A/2). The ± only appears when we take square roots of squared quantities.

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 5.2 Triple Angle Formulas](02-triple-angle.md) | [Unit 5 Index](README.md) | [5.4 Power Reduction Formulas →](04-power-reduction.md) |

---

[← Back to Main Index](../README.md)
