# Chapter 5.2: Triple Angle Formulas

## Overview

Triple angle formulas express trigonometric functions of **3A** in terms of functions of **A**. These are derived by treating 3A as 2A + A and applying the compound angle formulas along with double angle formulas.

---

## 📐 The Triple Angle Formulas

### Three Main Formulas

$$\boxed{\sin 3A = 3\sin A - 4\sin^3 A}$$

$$\boxed{\cos 3A = 4\cos^3 A - 3\cos A}$$

$$\boxed{\tan 3A = \frac{3\tan A - \tan^3 A}{1 - 3\tan^2 A}}$$

---

## 🔍 Derivations

### Deriving sin 3A

$$\sin 3A = \sin(2A + A)$$
$$= \sin 2A \cos A + \cos 2A \sin A$$

Substituting double angle formulas:
$$= (2\sin A \cos A)\cos A + (1 - 2\sin^2 A)\sin A$$
$$= 2\sin A \cos^2 A + \sin A - 2\sin^3 A$$

Using cos²A = 1 - sin²A:
$$= 2\sin A(1 - \sin^2 A) + \sin A - 2\sin^3 A$$
$$= 2\sin A - 2\sin^3 A + \sin A - 2\sin^3 A$$
$$= 3\sin A - 4\sin^3 A$$

### Deriving cos 3A

$$\cos 3A = \cos(2A + A)$$
$$= \cos 2A \cos A - \sin 2A \sin A$$

Substituting double angle formulas:
$$= (2\cos^2 A - 1)\cos A - (2\sin A \cos A)\sin A$$
$$= 2\cos^3 A - \cos A - 2\sin^2 A \cos A$$

Using sin²A = 1 - cos²A:
$$= 2\cos^3 A - \cos A - 2(1 - \cos^2 A)\cos A$$
$$= 2\cos^3 A - \cos A - 2\cos A + 2\cos^3 A$$
$$= 4\cos^3 A - 3\cos A$$

### Deriving tan 3A

$$\tan 3A = \tan(2A + A)$$
$$= \frac{\tan 2A + \tan A}{1 - \tan 2A \tan A}$$

Substituting tan 2A = 2tan A/(1 - tan²A):
$$= \frac{\frac{2\tan A}{1 - \tan^2 A} + \tan A}{1 - \frac{2\tan A}{1 - \tan^2 A} \cdot \tan A}$$

Multiply numerator and denominator by (1 - tan²A):
$$= \frac{2\tan A + \tan A(1 - \tan^2 A)}{(1 - \tan^2 A) - 2\tan^2 A}$$
$$= \frac{2\tan A + \tan A - \tan^3 A}{1 - \tan^2 A - 2\tan^2 A}$$
$$= \frac{3\tan A - \tan^3 A}{1 - 3\tan^2 A}$$

---

## 📊 Formula Reference Chart

```
    ┌─────────────────────────────────────────────────────────────────┐
    │                    TRIPLE ANGLE FORMULAS                        │
    ├─────────────────────────────────────────────────────────────────┤
    │                                                                 │
    │  sin 3A = 3 sin A - 4 sin³A                                    │
    │           ↓           ↓                                         │
    │         linear      cubic                                       │
    │                                                                 │
    │  cos 3A = 4 cos³A - 3 cos A                                    │
    │           ↓           ↓                                         │
    │         cubic       linear                                      │
    │                                                                 │
    │  tan 3A = (3 tan A - tan³A) / (1 - 3 tan²A)                    │
    │                                                                 │
    └─────────────────────────────────────────────────────────────────┘
```

---

## 🧠 Memory Techniques

### Coefficient Pattern

```
    sin 3A = 3 sin A - 4 sin³A
                ↓         ↓
    Coefficients: 3 and 4 (consecutive numbers!)
    
    cos 3A = 4 cos³A - 3 cos A
                ↓         ↓
    Coefficients: 4 and 3 (same numbers, reversed position!)
```

### "Sin starts Linear, Cos starts Cubic"

| Formula | First Term | Second Term |
|---------|------------|-------------|
| sin 3A | 3 sin A (linear) | -4 sin³A (cubic) |
| cos 3A | 4 cos³A (cubic) | -3 cos A (linear) |

### Sign Pattern
- Both formulas have **subtraction** between terms
- tan 3A: numerator has subtraction, denominator has subtraction

---

## 📐 Alternative Forms

### Factored Forms

$$\sin 3A = \sin A(3 - 4\sin^2 A)$$

$$\cos 3A = \cos A(4\cos^2 A - 3)$$

### Using Double Angle

$$\sin 3A = \sin A(1 + 2\cos 2A)$$

$$\cos 3A = \cos A(2\cos 2A - 1)$$

### Product Forms

$$\sin 3A = 4\sin A \sin(60° - A)\sin(60° + A)$$

$$\cos 3A = 4\cos A \cos(60° - A)\cos(60° + A)$$

$$\tan 3A = \tan A \tan(60° - A)\tan(60° + A)$$

---

## 🧮 Worked Examples

### Example 1: Finding sin 3A from sin A

If sin A = 1/2, find sin 3A.

**Solution:**
$$\sin 3A = 3\sin A - 4\sin^3 A$$
$$= 3 \cdot \frac{1}{2} - 4 \cdot \left(\frac{1}{2}\right)^3$$
$$= \frac{3}{2} - 4 \cdot \frac{1}{8}$$
$$= \frac{3}{2} - \frac{1}{2}$$
$$= \boxed{1}$$

**Verification:** If sin A = 1/2, then A = 30°, so 3A = 90°, and sin 90° = 1 ✓

### Example 2: Finding cos 3A from cos A

If cos A = 1/2, find cos 3A.

**Solution:**
$$\cos 3A = 4\cos^3 A - 3\cos A$$
$$= 4 \cdot \left(\frac{1}{2}\right)^3 - 3 \cdot \frac{1}{2}$$
$$= 4 \cdot \frac{1}{8} - \frac{3}{2}$$
$$= \frac{1}{2} - \frac{3}{2}$$
$$= \boxed{-1}$$

**Verification:** If cos A = 1/2, then A = 60°, so 3A = 180°, and cos 180° = -1 ✓

### Example 3: Finding tan 3A from tan A

If tan A = 1, find tan 3A.

**Solution:**
$$\tan 3A = \frac{3\tan A - \tan^3 A}{1 - 3\tan^2 A}$$
$$= \frac{3(1) - 1^3}{1 - 3(1)^2}$$
$$= \frac{3 - 1}{1 - 3}$$
$$= \frac{2}{-2}$$
$$= \boxed{-1}$$

**Verification:** If tan A = 1, then A = 45°, so 3A = 135°, and tan 135° = -1 ✓

### Example 4: Proving an Identity

Prove that $\sin 3A + \cos 3A = (\cos A - \sin A)(1 + 4\sin A \cos A)$

**Solution:**
$$\text{LHS} = \sin 3A + \cos 3A$$
$$= (3\sin A - 4\sin^3 A) + (4\cos^3 A - 3\cos A)$$
$$= 3(\sin A - \cos A) - 4(\sin^3 A - \cos^3 A)$$

Using a³ - b³ = (a - b)(a² + ab + b²):
$$= 3(\sin A - \cos A) - 4(\sin A - \cos A)(\sin^2 A + \sin A \cos A + \cos^2 A)$$
$$= (\sin A - \cos A)[3 - 4(1 + \sin A \cos A)]$$
$$= (\sin A - \cos A)(3 - 4 - 4\sin A \cos A)$$
$$= (\sin A - \cos A)(-1 - 4\sin A \cos A)$$
$$= -(\sin A - \cos A)(1 + 4\sin A \cos A)$$
$$= (\cos A - \sin A)(1 + 4\sin A \cos A) = \text{RHS} \quad \checkmark$$

### Example 5: Solving an Equation

Solve: sin 3θ = 4sin θ cos θ sin 2θ for θ ∈ [0, 2π]

**Solution:**
$$\sin 3\theta = 4\sin\theta \cos\theta \cdot 2\sin\theta\cos\theta$$
$$3\sin\theta - 4\sin^3\theta = 8\sin^2\theta\cos^2\theta$$
$$\sin\theta(3 - 4\sin^2\theta) = 8\sin^2\theta\cos^2\theta$$

If sin θ ≠ 0:
$$3 - 4\sin^2\theta = 8\sin\theta\cos^2\theta$$

This becomes complex. Let's try sin θ = 0:
$$\sin\theta = 0 \Rightarrow \theta = 0, \pi, 2\pi$$

### Example 6: Express cos 3A in Terms of cos 2A

Express cos 3A in terms of cos A and cos 2A.

**Solution:**
$$\cos 3A = \cos(2A + A)$$
$$= \cos 2A \cos A - \sin 2A \sin A$$
$$= \cos 2A \cos A - 2\sin A \cos A \cdot \sin A$$
$$= \cos 2A \cos A - 2\sin^2 A \cos A$$
$$= \cos A(\cos 2A - 2\sin^2 A)$$
$$= \cos A(\cos 2A - (1 - \cos 2A))$$
$$= \boxed{\cos A(2\cos 2A - 1)}$$

---

## 📊 Relationship with Standard Angles

### Values Check Table

| A | sin A | sin 3A (calculated) | sin 3A (direct) |
|---|-------|---------------------|-----------------|
| 0° | 0 | 3(0) - 4(0) = 0 | sin 0° = 0 ✓ |
| 30° | 1/2 | 3/2 - 4/8 = 1 | sin 90° = 1 ✓ |
| 45° | √2/2 | 3√2/2 - 4(√2/2)³ = √2/2 | sin 135° = √2/2 ✓ |
| 60° | √3/2 | 3√3/2 - 4(3√3/8) = 0 | sin 180° = 0 ✓ |

---

## 📐 Visualization

### sin 3A Curve Comparison

```
    y
    │     sin A (gentle)           sin 3A (oscillates 3× faster)
    │   
  1 ┤        ___                        __    __    __
    │       /   \                      /  \  /  \  /  \
    │      /     \                    /    \/    \/    \
  0 ┼─────/───────\─────────x        /────────────────────x
    │    /         \                /
    │   /           \               ‾‾    ‾‾    ‾‾
 -1 ┤  /             \___          
    │
    └──────────────────────
        0    π/2    π            0   π/3  2π/3   π
```

---

## 🌍 Applications

### 1. Cubic Equations
The equation 4x³ - 3x = cos 3A is solved by x = cos A
This is used to solve cubic equations of the form ax³ + bx + c = 0

### 2. Signal Processing
Triple frequency generation in electronics

### 3. Fourier Analysis
Expressing periodic functions as combinations of sine and cosine waves

### 4. Chebyshev Polynomials
cos nA = Tₙ(cos A) where T₃(x) = 4x³ - 3x

---

## 📋 Summary Table

### Formula Quick Reference

| Formula | Expression |
|---------|------------|
| sin 3A | 3 sin A - 4 sin³A |
| cos 3A | 4 cos³A - 3 cos A |
| tan 3A | (3 tan A - tan³A)/(1 - 3 tan²A) |

### Factored Forms

| Formula | Factored Expression |
|---------|---------------------|
| sin 3A | sin A(3 - 4sin²A) |
| cos 3A | cos A(4cos²A - 3) |
| sin 3A | 4 sin A sin(60°-A) sin(60°+A) |
| cos 3A | 4 cos A cos(60°-A) cos(60°+A) |

### Key Points

| Concept | Detail |
|---------|--------|
| Coefficients | 3 and 4 (consecutive) |
| sin 3A | Linear term first |
| cos 3A | Cubic term first |
| Derivation method | Use 3A = 2A + A |
| Zero of sin 3A | When sin A = 0 or sin²A = 3/4 |
| Zero of cos 3A | When cos A = 0 or cos²A = 3/4 |

---

## ❓ Quick Revision Questions

1. **If sin A = 1/3, find sin 3A.**

2. **If cos A = 2/3, find cos 3A.**

3. **Prove that sin 3A/sin A - cos 3A/cos A = 2**

4. **Express 4 cos³A - 3 cos A in terms of a single cosine function.**

5. **If tan A = 1/2, find tan 3A.**

6. **Why are the triple angle formulas useful for solving cubic equations?**

<details>
<summary>Click to see answers</summary>

1. sin 3A = 3(1/3) - 4(1/3)³  
   = 1 - 4/27  
   = 27/27 - 4/27  
   = **23/27**

2. cos 3A = 4(2/3)³ - 3(2/3)  
   = 4(8/27) - 2  
   = 32/27 - 54/27  
   = **-22/27**

3. LHS = sin 3A/sin A - cos 3A/cos A  
   = (3 sin A - 4sin³A)/sin A - (4cos³A - 3cos A)/cos A  
   = 3 - 4sin²A - 4cos²A + 3  
   = 6 - 4(sin²A + cos²A)  
   = 6 - 4(1)  
   = **2** ✓

4. 4cos³A - 3cos A = **cos 3A**

5. tan 3A = (3(1/2) - (1/2)³)/(1 - 3(1/4))  
   = (3/2 - 1/8)/(1 - 3/4)  
   = (12/8 - 1/8)/(1/4)  
   = (11/8) × 4  
   = **11/2**

6. Consider the cubic equation ax³ + bx + c = 0.  
   If we can write it in the form 4x³ - 3x = k where |k| ≤ 1,  
   then substituting x = cos A gives cos 3A = k.  
   So 3A = arccos(k), and x = cos(arccos(k)/3).  
   This provides a trigonometric solution to certain cubic equations.

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 5.1 Double Angle Formulas](01-double-angle.md) | [Unit 5 Index](README.md) | [5.3 Half Angle Formulas →](03-half-angle.md) |

---

[← Back to Main Index](../README.md)
