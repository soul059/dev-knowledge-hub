# Chapter 5.1: Double Angle Formulas

## Overview

Double angle formulas express trigonometric functions of **2A** in terms of functions of **A**. They are derived directly from the compound angle formulas by setting B = A.

---

## 📐 The Double Angle Formulas

### Three Main Formulas

$$\boxed{\sin 2A = 2\sin A \cos A}$$

$$\boxed{\cos 2A = \cos^2 A - \sin^2 A = 2\cos^2 A - 1 = 1 - 2\sin^2 A}$$

$$\boxed{\tan 2A = \frac{2\tan A}{1 - \tan^2 A}}$$

---

## 🔍 Derivations

### Deriving sin 2A

Start with the addition formula:
$$\sin(A + B) = \sin A \cos B + \cos A \sin B$$

Set B = A:
$$\sin(A + A) = \sin A \cos A + \cos A \sin A$$
$$\sin 2A = 2\sin A \cos A$$

### Deriving cos 2A (Form 1)

Start with:
$$\cos(A + B) = \cos A \cos B - \sin A \sin B$$

Set B = A:
$$\cos(A + A) = \cos A \cos A - \sin A \sin A$$
$$\cos 2A = \cos^2 A - \sin^2 A$$

### Deriving cos 2A (Form 2)

From Form 1, using sin²A = 1 - cos²A:
$$\cos 2A = \cos^2 A - (1 - \cos^2 A)$$
$$\cos 2A = \cos^2 A - 1 + \cos^2 A$$
$$\cos 2A = 2\cos^2 A - 1$$

### Deriving cos 2A (Form 3)

From Form 1, using cos²A = 1 - sin²A:
$$\cos 2A = (1 - \sin^2 A) - \sin^2 A$$
$$\cos 2A = 1 - 2\sin^2 A$$

### Deriving tan 2A

Start with:
$$\tan(A + B) = \frac{\tan A + \tan B}{1 - \tan A \tan B}$$

Set B = A:
$$\tan 2A = \frac{\tan A + \tan A}{1 - \tan A \cdot \tan A}$$
$$\tan 2A = \frac{2\tan A}{1 - \tan^2 A}$$

---

## 📊 Formula Reference Chart

```
    ┌─────────────────────────────────────────────────────────────────┐
    │                    DOUBLE ANGLE FORMULAS                        │
    ├─────────────────────────────────────────────────────────────────┤
    │                                                                 │
    │  sin 2A = 2 sin A cos A                                        │
    │           └────┬────┘                                          │
    │           "twice the product"                                   │
    │                                                                 │
    │  ┌─────────────────────────────────────────────────────────┐   │
    │  │               THREE FORMS OF cos 2A                     │   │
    │  ├─────────────────────────────────────────────────────────┤   │
    │  │  cos 2A = cos²A - sin²A     (both functions)            │   │
    │  │         = 2cos²A - 1         (cosine only)              │   │
    │  │         = 1 - 2sin²A         (sine only)                │   │
    │  └─────────────────────────────────────────────────────────┘   │
    │                                                                 │
    │  tan 2A = 2 tan A / (1 - tan²A)                                │
    │                                                                 │
    └─────────────────────────────────────────────────────────────────┘
```

---

## 🧠 Memory Techniques

### sin 2A
- "**Two** sin cos" - multiply sin and cos by **2**
- Think: "Double = 2 × sin × cos"

### cos 2A - Choosing the Right Form

```
    ┌───────────────────────────────────────────────────────────────┐
    │  WHICH FORM TO USE?                                          │
    ├───────────────────────────────────────────────────────────────┤
    │                                                               │
    │  cos²A - sin²A  →  When you have BOTH sin A and cos A        │
    │                                                               │
    │  2cos²A - 1     →  When you only have cos A                  │
    │                 →  Or need to express cos²A in terms of cos2A│
    │                                                               │
    │  1 - 2sin²A     →  When you only have sin A                  │
    │                 →  Or need to express sin²A in terms of cos2A│
    │                                                               │
    └───────────────────────────────────────────────────────────────┘
```

### tan 2A
- "**2** tan over **1** minus tan²"
- Numerator: 2 tan A
- Denominator: 1 - tan²A

---

## 📈 Rearranged Forms (Useful for Integration)

From cos 2A = 2cos²A - 1:
$$\boxed{\cos^2 A = \frac{1 + \cos 2A}{2}}$$

From cos 2A = 1 - 2sin²A:
$$\boxed{\sin^2 A = \frac{1 - \cos 2A}{2}}$$

These are called **power reduction formulas** (covered in detail in Chapter 5.4).

---

## 🧮 Worked Examples

### Example 1: Finding sin 2A and cos 2A

If sin A = 3/5 and A is in the first quadrant, find sin 2A and cos 2A.

**Solution:**
First, find cos A using Pythagorean identity:
$$\cos A = \sqrt{1 - \sin^2 A} = \sqrt{1 - \frac{9}{25}} = \sqrt{\frac{16}{25}} = \frac{4}{5}$$

$$\sin 2A = 2\sin A \cos A = 2 \cdot \frac{3}{5} \cdot \frac{4}{5} = \boxed{\frac{24}{25}}$$

$$\cos 2A = \cos^2 A - \sin^2 A = \frac{16}{25} - \frac{9}{25} = \boxed{\frac{7}{25}}$$

### Example 2: Using tan 2A Formula

If tan A = 2, find tan 2A.

**Solution:**
$$\tan 2A = \frac{2\tan A}{1 - \tan^2 A} = \frac{2(2)}{1 - 4} = \frac{4}{-3} = \boxed{-\frac{4}{3}}$$

### Example 3: Express in Terms of Single Angle

Simplify: 1 - 2sin²(15°)

**Solution:**
Using 1 - 2sin²A = cos 2A:
$$1 - 2\sin^2(15°) = \cos(2 \times 15°) = \cos 30° = \boxed{\frac{\sqrt{3}}{2}}$$

### Example 4: Finding Values from tan A

If tan A = 1/2, find sin 2A and cos 2A.

**Solution:**
$$\tan 2A = \frac{2 \times \frac{1}{2}}{1 - \frac{1}{4}} = \frac{1}{\frac{3}{4}} = \frac{4}{3}$$

To find sin 2A and cos 2A from tan 2A = 4/3:

Draw a right triangle with opposite = 4, adjacent = 3:
- Hypotenuse = 5

$$\sin 2A = \frac{4}{5}, \quad \cos 2A = \frac{3}{5}$$

### Example 5: Proving an Identity

Prove that $\frac{\sin 2A}{1 + \cos 2A} = \tan A$

**Solution:**
$$\text{LHS} = \frac{\sin 2A}{1 + \cos 2A}$$
$$= \frac{2\sin A \cos A}{1 + (2\cos^2 A - 1)}$$
$$= \frac{2\sin A \cos A}{2\cos^2 A}$$
$$= \frac{\sin A}{\cos A}$$
$$= \tan A = \text{RHS} \quad \checkmark$$

### Example 6: Another Identity

Prove that $\frac{1 - \cos 2A}{\sin 2A} = \tan A$

**Solution:**
$$\text{LHS} = \frac{1 - \cos 2A}{\sin 2A}$$
$$= \frac{1 - (1 - 2\sin^2 A)}{2\sin A \cos A}$$
$$= \frac{2\sin^2 A}{2\sin A \cos A}$$
$$= \frac{\sin A}{\cos A}$$
$$= \tan A = \text{RHS} \quad \checkmark$$

---

## 📐 Visual Representation

### sin 2A Geometry

```
    In a right triangle with angle A:
    
           /|
          / |
         /  |
      1 /   | sin A
       /    |
      / A   |
     /______|
      cos A
    
    sin 2A = 2(sin A)(cos A)
           = 2 × (opposite/hyp) × (adjacent/hyp)
           = 2 × (area factor)
    
    This equals twice the area of the right triangle!
```

### cos 2A Visual

```
    cos 2A represents the horizontal displacement 
    when the angle is DOUBLED:
    
    Original angle A:      Double angle 2A:
    
         |                      |
         | .                    |   .
         |   .                  |       .
    ─────*─────                 *───────────
         cos A               cos 2A (can be negative!)
```

---

## 🌍 Applications

### 1. Physics: Simple Harmonic Motion
The kinetic energy in SHM: $KE = \frac{1}{2}m\omega^2 A^2\sin^2(\omega t) = \frac{1}{4}m\omega^2 A^2(1 - \cos 2\omega t)$

### 2. Electrical Engineering
AC power calculations: $P = VI\sin^2(\omega t) = \frac{VI}{2}(1 - \cos 2\omega t)$

### 3. Calculus Integration
$$\int \sin^2 x \, dx = \int \frac{1 - \cos 2x}{2} \, dx = \frac{x}{2} - \frac{\sin 2x}{4} + C$$

### 4. Optics
Double-slit interference patterns use these formulas.

---

## 📋 Summary Table

### Formula Quick Reference

| Formula | Expression |
|---------|------------|
| sin 2A | 2 sin A cos A |
| cos 2A (Form 1) | cos²A - sin²A |
| cos 2A (Form 2) | 2cos²A - 1 |
| cos 2A (Form 3) | 1 - 2sin²A |
| tan 2A | 2tan A/(1 - tan²A) |

### Rearranged Forms

| Expression | Equivalent |
|------------|------------|
| sin²A | (1 - cos 2A)/2 |
| cos²A | (1 + cos 2A)/2 |
| sin A cos A | sin 2A/2 |

### Key Points

| Concept | Detail |
|---------|--------|
| Source | Derived from compound angle formulas |
| Usage | Simplification, integration, equation solving |
| cos 2A forms | Choose based on available information |
| tan 2A undefined | When tan²A = 1 (A = 45°, 135°, etc.) |

---

## ❓ Quick Revision Questions

1. **If cos A = 5/13 and A is in Q1, find sin 2A and cos 2A.**

2. **Prove that: (sin A + cos A)² = 1 + sin 2A**

3. **Simplify: cos⁴A - sin⁴A**

4. **If tan θ = 3/4, find tan 2θ.**

5. **Prove that: $\frac{\sin 2A}{1 - \cos 2A} = \cot A$**

6. **Express sin 3A in terms of sin A (hint: use sin 3A = sin(2A + A)).**

<details>
<summary>Click to see answers</summary>

1. If cos A = 5/13 in Q1:
   sin A = √(1 - 25/169) = √(144/169) = 12/13  
   sin 2A = 2(12/13)(5/13) = **120/169**  
   cos 2A = (5/13)² - (12/13)² = 25/169 - 144/169 = **-119/169**

2. LHS = sin²A + 2 sin A cos A + cos²A  
   = 1 + 2 sin A cos A  
   = 1 + sin 2A = RHS ✓

3. cos⁴A - sin⁴A = (cos²A + sin²A)(cos²A - sin²A)  
   = 1 · cos 2A = **cos 2A**

4. tan 2θ = 2(3/4)/(1 - 9/16)  
   = (3/2)/(7/16)  
   = (3/2) × (16/7)  
   = **24/7**

5. LHS = sin 2A/(1 - cos 2A)  
   = 2 sin A cos A/(1 - (1 - 2sin²A))  
   = 2 sin A cos A/(2sin²A)  
   = cos A/sin A = **cot A** ✓

6. sin 3A = sin(2A + A)  
   = sin 2A cos A + cos 2A sin A  
   = 2 sin A cos A · cos A + (1 - 2sin²A) sin A  
   = 2 sin A cos²A + sin A - 2sin³A  
   = 2 sin A(1 - sin²A) + sin A - 2sin³A  
   = 2 sin A - 2sin³A + sin A - 2sin³A  
   = **3 sin A - 4sin³A**

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Unit 4: Applications](../04-Compound-Angles/04-applications.md) | [Unit 5 Index](README.md) | [5.2 Triple Angle Formulas →](02-triple-angle.md) |

---

[← Back to Main Index](../README.md)
