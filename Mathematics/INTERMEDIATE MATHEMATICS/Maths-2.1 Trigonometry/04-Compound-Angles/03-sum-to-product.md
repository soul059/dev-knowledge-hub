# Chapter 4.3: Sum to Product Formulas

## Overview

Sum to product formulas are the **reverse** of product to sum formulas. They convert **sums or differences** of trigonometric functions into **products**. These are especially useful for solving equations, simplifying expressions, and proving identities.

---

## 📐 The Sum to Product Formulas

### The Four Formulas

$$\boxed{\sin A + \sin B = 2\sin\left(\frac{A + B}{2}\right)\cos\left(\frac{A - B}{2}\right)}$$

$$\boxed{\sin A - \sin B = 2\cos\left(\frac{A + B}{2}\right)\sin\left(\frac{A - B}{2}\right)}$$

$$\boxed{\cos A + \cos B = 2\cos\left(\frac{A + B}{2}\right)\cos\left(\frac{A - B}{2}\right)}$$

$$\boxed{\cos A - \cos B = -2\sin\left(\frac{A + B}{2}\right)\sin\left(\frac{A - B}{2}\right)}$$

---

## 🔍 Derivation of Sum to Product Formulas

### Key Substitution

Let $A = P + Q$ and $B = P - Q$

Then:
$$P = \frac{A + B}{2} \quad \text{and} \quad Q = \frac{A - B}{2}$$

### Deriving sin A + sin B

Start with product to sum formula:
$$2\sin P \cos Q = \sin(P + Q) + \sin(P - Q)$$

Substituting P + Q = A and P - Q = B:
$$2\sin\left(\frac{A+B}{2}\right)\cos\left(\frac{A-B}{2}\right) = \sin A + \sin B$$

$$\therefore \sin A + \sin B = 2\sin\left(\frac{A+B}{2}\right)\cos\left(\frac{A-B}{2}\right)$$

### Deriving sin A - sin B

From:
$$2\cos P \sin Q = \sin(P + Q) - \sin(P - Q)$$

$$\therefore \sin A - \sin B = 2\cos\left(\frac{A+B}{2}\right)\sin\left(\frac{A-B}{2}\right)$$

### Deriving cos A + cos B

From:
$$2\cos P \cos Q = \cos(P + Q) + \cos(P - Q)$$

$$\therefore \cos A + \cos B = 2\cos\left(\frac{A+B}{2}\right)\cos\left(\frac{A-B}{2}\right)$$

### Deriving cos A - cos B

From:
$$-2\sin P \sin Q = \cos(P + Q) - \cos(P - Q)$$

$$\therefore \cos A - \cos B = -2\sin\left(\frac{A+B}{2}\right)\sin\left(\frac{A-B}{2}\right)$$

---

## 📊 Visual Formula Chart

```
    ┌────────────────────────────────────────────────────────────────────┐
    │                    SUM TO PRODUCT FORMULAS                         │
    ├────────────────────────────────────────────────────────────────────┤
    │                                                                    │
    │  sin A + sin B = 2 sin((A+B)/2) cos((A-B)/2)    [sin + sin]       │
    │                      ↓              ↓                              │
    │                   "sum"          "diff"                            │
    │                                                                    │
    │  sin A - sin B = 2 cos((A+B)/2) sin((A-B)/2)    [sin - sin]       │
    │                      ↓              ↓                              │
    │                   "sum"          "diff"                            │
    │                                                                    │
    │  cos A + cos B = 2 cos((A+B)/2) cos((A-B)/2)    [cos + cos]       │
    │                      ↓              ↓                              │
    │                   "sum"          "diff"                            │
    │                                                                    │
    │  cos A - cos B = -2 sin((A+B)/2) sin((A-B)/2)   [cos - cos]       │
    │                 ↑                                                  │
    │            NEGATIVE!                                               │
    │                                                                    │
    └────────────────────────────────────────────────────────────────────┘
```

---

## 🧠 Memory Techniques

### The "Half-Angle" Pattern

```
    Both arguments use HALF of the original angles:
    
    ┌─────────────┐     ┌─────────────┐
    │   A + B     │     │   A - B     │
    │  ───────    │     │  ───────    │
    │     2       │     │     2       │
    └─────────────┘     └─────────────┘
         ↓                   ↓
       "Sum"              "Diff"
      (average)          (half-diff)
```

### The "SSCC" Memory Rule

For the **first function** in the product:

| Expression | First Function | Second Function |
|------------|----------------|-----------------|
| sin + sin | **S**in | cos |
| sin - sin | **C**os | sin |
| cos + cos | **C**os | cos |
| cos - cos | **S**in (negative!) | sin |

Pattern: **S-C-C-S** going down!

### Sign Rule

- Only **cos A - cos B** has a **negative** sign
- Remember: "**C**osine **D**ifference is **D**ifferent" (negative)

---

## 🧮 Worked Examples

### Example 1: sin + sin

Express sin 5x + sin 3x as a product.

**Solution:**
Using sin A + sin B = 2 sin((A+B)/2) cos((A-B)/2):

Here A = 5x, B = 3x
$$\frac{A + B}{2} = \frac{5x + 3x}{2} = 4x$$
$$\frac{A - B}{2} = \frac{5x - 3x}{2} = x$$

$$\sin 5x + \sin 3x = 2\sin 4x \cos x$$

### Example 2: sin - sin

Express sin 7θ - sin 3θ as a product.

**Solution:**
Using sin A - sin B = 2 cos((A+B)/2) sin((A-B)/2):

Here A = 7θ, B = 3θ
$$\frac{A + B}{2} = \frac{7\theta + 3\theta}{2} = 5\theta$$
$$\frac{A - B}{2} = \frac{7\theta - 3\theta}{2} = 2\theta$$

$$\sin 7\theta - \sin 3\theta = 2\cos 5\theta \sin 2\theta$$

### Example 3: cos + cos

Express cos 6x + cos 2x as a product.

**Solution:**
Using cos A + cos B = 2 cos((A+B)/2) cos((A-B)/2):

Here A = 6x, B = 2x
$$\frac{A + B}{2} = \frac{6x + 2x}{2} = 4x$$
$$\frac{A - B}{2} = \frac{6x - 2x}{2} = 2x$$

$$\cos 6x + \cos 2x = 2\cos 4x \cos 2x$$

### Example 4: cos - cos

Express cos 5θ - cos θ as a product.

**Solution:**
Using cos A - cos B = -2 sin((A+B)/2) sin((A-B)/2):

Here A = 5θ, B = θ
$$\frac{A + B}{2} = \frac{5\theta + \theta}{2} = 3\theta$$
$$\frac{A - B}{2} = \frac{5\theta - \theta}{2} = 2\theta$$

$$\cos 5\theta - \cos \theta = -2\sin 3\theta \sin 2\theta$$

### Example 5: Numerical Calculation

Find the exact value of sin 75° + sin 15°.

**Solution:**
$$\sin 75° + \sin 15° = 2\sin\left(\frac{75° + 15°}{2}\right)\cos\left(\frac{75° - 15°}{2}\right)$$
$$= 2\sin 45° \cos 30°$$
$$= 2 \cdot \frac{\sqrt{2}}{2} \cdot \frac{\sqrt{3}}{2}$$
$$= \frac{\sqrt{6}}{2}$$

### Example 6: Proving an Identity

Prove that $\frac{\sin A + \sin B}{\cos A + \cos B} = \tan\left(\frac{A+B}{2}\right)$

**Solution:**
$$\text{LHS} = \frac{\sin A + \sin B}{\cos A + \cos B}$$

$$= \frac{2\sin\left(\frac{A+B}{2}\right)\cos\left(\frac{A-B}{2}\right)}{2\cos\left(\frac{A+B}{2}\right)\cos\left(\frac{A-B}{2}\right)}$$

$$= \frac{\sin\left(\frac{A+B}{2}\right)}{\cos\left(\frac{A+B}{2}\right)}$$

$$= \tan\left(\frac{A+B}{2}\right) = \text{RHS} \quad \checkmark$$

---

## 📐 Special Identities Derived

### Important Results

```
    ┌────────────────────────────────────────────────────────────────┐
    │                    DERIVED IDENTITIES                          │
    ├────────────────────────────────────────────────────────────────┤
    │                                                                │
    │  sin A + sin B                    A + B                        │
    │  ───────────────  =  tan  ( ─────────── )                      │
    │  cos A + cos B                    2                            │
    │                                                                │
    │  sin A - sin B                    A - B                        │
    │  ───────────────  =  tan  ( ─────────── )                      │
    │  cos A + cos B                    2                            │
    │                                                                │
    │  sin A + sin B                    A + B                        │
    │  ───────────────  =  cot  ( ─────────── )                      │
    │  cos A - cos B                    2                            │
    │         ↑                                                      │
    │    (negative in cos-cos formula)                               │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 🌍 Applications

### 1. Solving Trigonometric Equations

The equation sin 5x + sin x = 0 becomes:
$$2\sin 3x \cos 2x = 0$$
$$\sin 3x = 0 \quad \text{or} \quad \cos 2x = 0$$

Much easier to solve!

### 2. Beat Frequencies in Physics

When two sound waves of frequencies f₁ and f₂ combine:
$$y = \sin(2\pi f_1 t) + \sin(2\pi f_2 t)$$
$$= 2\sin\left(\pi(f_1 + f_2)t\right)\cos\left(\pi(f_1 - f_2)t\right)$$

The beat frequency = |f₁ - f₂|

### 3. Simplifying Series

Sums like sin θ + sin 2θ + sin 3θ + ... can be simplified using these formulas.

---

## 📊 Comparison: Product ↔ Sum

```
    PRODUCT TO SUM                    SUM TO PRODUCT
    ──────────────                    ──────────────
    2 sin A cos B                     sin A + sin B
         ↓                                  ↓
    sin(A+B) + sin(A-B)               2 sin((A+B)/2) cos((A-B)/2)
    
    ────────────────────────────────────────────────────────────
                        INVERSE OPERATIONS
```

---

## 📋 Summary Table

### Quick Reference

| Sum/Difference | Product Form |
|----------------|--------------|
| sin A + sin B | 2 sin((A+B)/2) cos((A-B)/2) |
| sin A - sin B | 2 cos((A+B)/2) sin((A-B)/2) |
| cos A + cos B | 2 cos((A+B)/2) cos((A-B)/2) |
| cos A - cos B | **-**2 sin((A+B)/2) sin((A-B)/2) |

### Key Points

| Concept | Details |
|---------|---------|
| Arguments | Always (A+B)/2 and (A-B)/2 |
| Factor 2 | Always present in product |
| Negative sign | Only in cos A - cos B formula |
| sin ± sin | Results have both sin and cos |
| cos ± cos | Results have sin·sin or cos·cos |

---

## 📈 Process Diagram

```
    Given: sin A ± sin B  OR  cos A ± cos B
    
    Step 1: Calculate (A + B)/2
            └─→ This goes in the FIRST function
    
    Step 2: Calculate (A - B)/2
            └─→ This goes in the SECOND function
    
    Step 3: Apply the appropriate formula:
            ┌─ sin + sin → 2 sin cos
            ├─ sin - sin → 2 cos sin
            ├─ cos + cos → 2 cos cos
            └─ cos - cos → -2 sin sin
    
    Step 4: Simplify if possible
```

---

## ❓ Quick Revision Questions

1. **Express sin 8x + sin 2x as a product.**

2. **Express cos 7θ - cos 3θ as a product.**

3. **Express sin 40° - sin 20° as a product.**

4. **Find the exact value of cos 75° + cos 15°.**

5. **Prove that $\frac{\sin 3A - \sin A}{\cos A - \cos 3A} = \cot 2A$**

6. **Why does the formula for cos A - cos B have a negative sign while others don't?**

<details>
<summary>Click to see answers</summary>

1. sin 8x + sin 2x = 2 sin 5x cos 3x  
   (A+B)/2 = 5x, (A-B)/2 = 3x

2. cos 7θ - cos 3θ = -2 sin 5θ sin 2θ  
   (A+B)/2 = 5θ, (A-B)/2 = 2θ

3. sin 40° - sin 20° = 2 cos 30° sin 10°  
   = 2 · (√3/2) · sin 10°  
   = **√3 sin 10°**

4. cos 75° + cos 15° = 2 cos 45° cos 30°  
   = 2 · (√2/2) · (√3/2)  
   = **√6/2**

5. LHS = (sin 3A - sin A)/(cos A - cos 3A)  
   = [2 cos 2A sin A] / [-2 sin 2A sin(-A)]  
   = [2 cos 2A sin A] / [2 sin 2A sin A]  
   = cos 2A / sin 2A  
   = **cot 2A** = RHS ✓

6. It comes from the derivation. When we subtract cosines:
   cos(P+Q) - cos(P-Q) = -2 sin P sin Q  
   The negative appears because:  
   cos(P+Q) = cos P cos Q - sin P sin Q  
   cos(P-Q) = cos P cos Q + sin P sin Q  
   Subtracting: -2 sin P sin Q (the cos terms cancel, sin terms have opposite signs)

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 4.2 Product to Sum](02-product-to-sum.md) | [Unit 4 Index](README.md) | [4.4 Applications →](04-applications.md) |

---

[← Back to Main Index](../README.md)
