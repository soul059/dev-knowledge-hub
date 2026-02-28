# Chapter 4.1: Addition and Subtraction Formulas

## Overview

The addition and subtraction formulas (also called compound angle formulas) express the trigonometric functions of a sum or difference of angles in terms of the functions of individual angles. These are among the most important formulas in trigonometry.

---

## 📐 The Sine Addition and Subtraction Formulas

### Formulas

$$\boxed{\sin(A + B) = \sin A \cos B + \cos A \sin B}$$

$$\boxed{\sin(A - B) = \sin A \cos B - \cos A \sin B}$$

### Combined Form

$$\sin(A \pm B) = \sin A \cos B \pm \cos A \sin B$$

### Geometric Proof for sin(A + B)

```
                        P
                       /|
                      / |
                     /  |
                    /   |
                   /    | sin(A+B)
                  / A+B |
                 /______|___________
                O       Q           R
                
    Consider point P on unit circle at angle (A+B)
    
    Alternative construction:
    
                     D
                    /|
                   / |
                  /  |E
                 /   |
                / A  |        sin(A+B) = DE + EC
               /_____|____               = sin A cos B + cos A sin B
              O   C   B
                   \
                    \ B
                     \
                      F
```

**Proof using unit circle and projections:**

Let two angles A and B be given. On the unit circle:
- For angle A: point has coordinates (cos A, sin A)
- For angle B: we need to rotate by B

Using rotation matrix or geometric projection:
$$\sin(A + B) = \sin A \cos B + \cos A \sin B$$

---

## 📐 The Cosine Addition and Subtraction Formulas

### Formulas

$$\boxed{\cos(A + B) = \cos A \cos B - \sin A \sin B}$$

$$\boxed{\cos(A - B) = \cos A \cos B + \sin A \sin B}$$

### Combined Form

$$\cos(A \pm B) = \cos A \cos B \mp \sin A \sin B$$

**Note:** The sign in the middle is **opposite** to the sign between A and B.

### Proof for cos(A - B) using Distance Formula

```
    Unit Circle with two points:
    
    P₁ at angle A: (cos A, sin A)
    P₂ at angle B: (cos B, sin B)
    
                    P₁(cos A, sin A)
                   ●
                  /│ angle A
                 / │
                /  │
               /   │
              /    │
    ─────────●─────┼─────────
            O│     │
              \    │
               \   │
                \  │
                 \ │ angle B
                  \│
                   ●
                    P₂(cos B, sin B)
                    
    Distance P₁P₂ can be calculated two ways:
    1. Using coordinates
    2. Using angle (A - B)
```

**Step 1:** Distance between P₁ and P₂ using coordinates:
$$d² = (\cos A - \cos B)² + (\sin A - \sin B)²$$
$$= \cos²A - 2\cos A\cos B + \cos²B + \sin²A - 2\sin A\sin B + \sin²B$$
$$= (\cos²A + \sin²A) + (\cos²B + \sin²B) - 2(\cos A\cos B + \sin A\sin B)$$
$$= 1 + 1 - 2(\cos A\cos B + \sin A\sin B)$$
$$= 2 - 2(\cos A\cos B + \sin A\sin B)$$

**Step 2:** Same distance using the angle between them:
If Q is at angle (A-B) from (1, 0):
$$d² = (\cos(A-B) - 1)² + \sin²(A-B)$$
$$= \cos²(A-B) - 2\cos(A-B) + 1 + \sin²(A-B)$$
$$= 1 - 2\cos(A-B) + 1$$
$$= 2 - 2\cos(A-B)$$

**Step 3:** Equating:
$$2 - 2(\cos A\cos B + \sin A\sin B) = 2 - 2\cos(A-B)$$
$$\cos(A-B) = \cos A\cos B + \sin A\sin B$$

---

## 📐 The Tangent Addition and Subtraction Formulas

### Formulas

$$\boxed{\tan(A + B) = \frac{\tan A + \tan B}{1 - \tan A \tan B}}$$

$$\boxed{\tan(A - B) = \frac{\tan A - \tan B}{1 + \tan A \tan B}}$$

### Combined Form

$$\tan(A \pm B) = \frac{\tan A \pm \tan B}{1 \mp \tan A \tan B}$$

### Derivation

$$\tan(A + B) = \frac{\sin(A + B)}{\cos(A + B)}$$

$$= \frac{\sin A \cos B + \cos A \sin B}{\cos A \cos B - \sin A \sin B}$$

Divide numerator and denominator by cos A cos B:

$$= \frac{\frac{\sin A}{\cos A} + \frac{\sin B}{\cos B}}{1 - \frac{\sin A \sin B}{\cos A \cos B}}$$

$$= \frac{\tan A + \tan B}{1 - \tan A \tan B}$$

---

## 📊 Complete Formula Reference

| Formula | Expression |
|---------|------------|
| sin(A + B) | sin A cos B + cos A sin B |
| sin(A - B) | sin A cos B - cos A sin B |
| cos(A + B) | cos A cos B - sin A sin B |
| cos(A - B) | cos A cos B + sin A sin B |
| tan(A + B) | (tan A + tan B)/(1 - tan A tan B) |
| tan(A - B) | (tan A - tan B)/(1 + tan A tan B) |

---

## 🧠 Memory Techniques

### For Sine

"**S**ine = **S**ame sign"

$$\sin(A \pm B) = \sin A \cos B \pm \cos A \sin B$$

The sign in the formula matches the sign between A and B.

### For Cosine

"**C**osine = **C**hange sign"

$$\cos(A \pm B) = \cos A \cos B \mp \sin A \sin B$$

The sign in the formula is opposite to the sign between A and B.

### For Tangent

"Top same, bottom opposite"

$$\tan(A \pm B) = \frac{\tan A \pm \tan B}{1 \mp \tan A \tan B}$$

---

## 🧮 Worked Examples

### Example 1: Finding Exact Values

Find the exact value of sin 75°.

**Solution:**
$$\sin 75° = \sin(45° + 30°)$$
$$= \sin 45° \cos 30° + \cos 45° \sin 30°$$
$$= \frac{\sqrt{2}}{2} \cdot \frac{\sqrt{3}}{2} + \frac{\sqrt{2}}{2} \cdot \frac{1}{2}$$
$$= \frac{\sqrt{6}}{4} + \frac{\sqrt{2}}{4}$$
$$= \boxed{\frac{\sqrt{6} + \sqrt{2}}{4}}$$

### Example 2: cos 15°

Find the exact value of cos 15°.

**Solution:**
$$\cos 15° = \cos(45° - 30°)$$
$$= \cos 45° \cos 30° + \sin 45° \sin 30°$$
$$= \frac{\sqrt{2}}{2} \cdot \frac{\sqrt{3}}{2} + \frac{\sqrt{2}}{2} \cdot \frac{1}{2}$$
$$= \frac{\sqrt{6}}{4} + \frac{\sqrt{2}}{4}$$
$$= \boxed{\frac{\sqrt{6} + \sqrt{2}}{4}}$$

### Example 3: tan 105°

Find the exact value of tan 105°.

**Solution:**
$$\tan 105° = \tan(60° + 45°)$$
$$= \frac{\tan 60° + \tan 45°}{1 - \tan 60° \tan 45°}$$
$$= \frac{\sqrt{3} + 1}{1 - \sqrt{3} \cdot 1}$$
$$= \frac{\sqrt{3} + 1}{1 - \sqrt{3}}$$

Rationalize by multiplying by $\frac{1 + \sqrt{3}}{1 + \sqrt{3}}$:
$$= \frac{(\sqrt{3} + 1)(1 + \sqrt{3})}{(1 - \sqrt{3})(1 + \sqrt{3})}$$
$$= \frac{\sqrt{3} + 3 + 1 + \sqrt{3}}{1 - 3}$$
$$= \frac{4 + 2\sqrt{3}}{-2}$$
$$= \boxed{-2 - \sqrt{3}}$$

### Example 4: Simplifying Expressions

Simplify: sin(x + 30°) + sin(x - 30°)

**Solution:**
$$= [\sin x \cos 30° + \cos x \sin 30°] + [\sin x \cos 30° - \cos x \sin 30°]$$
$$= 2 \sin x \cos 30°$$
$$= 2 \sin x \cdot \frac{\sqrt{3}}{2}$$
$$= \boxed{\sqrt{3} \sin x}$$

### Example 5: Proving an Identity

Prove: sin(A + B) sin(A - B) = sin²A - sin²B

**Proof:**
$$LHS = [\sin A \cos B + \cos A \sin B][\sin A \cos B - \cos A \sin B]$$

Using (a + b)(a - b) = a² - b²:
$$= (\sin A \cos B)² - (\cos A \sin B)²$$
$$= \sin²A \cos²B - \cos²A \sin²B$$
$$= \sin²A(1 - \sin²B) - (1 - \sin²A)\sin²B$$
$$= \sin²A - \sin²A\sin²B - \sin²B + \sin²A\sin²B$$
$$= \sin²A - \sin²B = RHS \quad \checkmark$$

### Example 6: Using Given Values

If sin A = 3/5 (A in Q1) and cos B = 5/13 (B in Q1), find sin(A + B).

**Solution:**

Find cos A: $\cos A = \sqrt{1 - 9/25} = 4/5$

Find sin B: $\sin B = \sqrt{1 - 25/169} = 12/13$

$$\sin(A + B) = \sin A \cos B + \cos A \sin B$$
$$= \frac{3}{5} \cdot \frac{5}{13} + \frac{4}{5} \cdot \frac{12}{13}$$
$$= \frac{15}{65} + \frac{48}{65}$$
$$= \boxed{\frac{63}{65}}$$

---

## 📊 Special Results from Compound Formulas

### Complementary Angle Relationships

Using A = 90° in subtraction formulas:
$$\sin(90° - B) = \sin 90° \cos B - \cos 90° \sin B = \cos B$$
$$\cos(90° - B) = \cos 90° \cos B + \sin 90° \sin B = \sin B$$

### Supplementary Angle Relationships

Using A = 180°:
$$\sin(180° - B) = \sin 180° \cos B - \cos 180° \sin B = \sin B$$
$$\cos(180° - B) = \cos 180° \cos B + \sin 180° \sin B = -\cos B$$

### Negative Angle Relationships

Using A = 0° in subtraction formulas:
$$\sin(-B) = \sin 0° \cos B - \cos 0° \sin B = -\sin B$$
$$\cos(-B) = \cos 0° \cos B + \sin 0° \sin B = \cos B$$

---

## 📐 Cotangent Formula (Bonus)

$$\cot(A + B) = \frac{\cot A \cot B - 1}{\cot A + \cot B}$$

$$\cot(A - B) = \frac{\cot A \cot B + 1}{\cot B - \cot A}$$

---

## 🌍 Real-World Applications

### 1. Signal Processing
Adding sound waves with different phases uses these formulas.

### 2. Physics - Wave Interference
Superposition of waves: $y_1 + y_2 = A\sin(\omega t + \phi_1) + A\sin(\omega t + \phi_2)$

### 3. Navigation
Calculating resultant directions when combining bearings.

### 4. Electrical Engineering
AC circuit analysis with phase differences.

---

## 📋 Summary Table

### All Addition/Subtraction Formulas

| Function | Addition | Subtraction |
|----------|----------|-------------|
| Sine | sin A cos B + cos A sin B | sin A cos B - cos A sin B |
| Cosine | cos A cos B - sin A sin B | cos A cos B + sin A sin B |
| Tangent | (tan A + tan B)/(1 - tan A tan B) | (tan A - tan B)/(1 + tan A tan B) |

### Common Exact Values

| Angle | Computed As | sin | cos | tan |
|-------|-------------|-----|-----|-----|
| 15° | 45° - 30° | (√6 - √2)/4 | (√6 + √2)/4 | 2 - √3 |
| 75° | 45° + 30° | (√6 + √2)/4 | (√6 - √2)/4 | 2 + √3 |
| 105° | 60° + 45° | (√6 + √2)/4 | (√2 - √6)/4 | -2 - √3 |

---

## ❓ Quick Revision Questions

1. **Write the formula for sin(A + B) from memory.**

2. **Find the exact value of cos 75°.**

3. **Simplify: cos(x + 60°) + cos(x - 60°)**

4. **If tan A = 1/2 and tan B = 1/3, find tan(A + B).**

5. **Prove that sin(A + B) + sin(A - B) = 2 sin A cos B.**

6. **Why is the sign in the cosine formula opposite to the sign between A and B?**

<details>
<summary>Click to see answers</summary>

1. sin(A + B) = sin A cos B + cos A sin B

2. cos 75° = cos(45° + 30°) = cos 45° cos 30° - sin 45° sin 30°  
   = (√2/2)(√3/2) - (√2/2)(1/2) = (√6 - √2)/4

3. cos(x + 60°) + cos(x - 60°)  
   = [cos x cos 60° - sin x sin 60°] + [cos x cos 60° + sin x sin 60°]  
   = 2 cos x cos 60° = 2 cos x · (1/2) = **cos x**

4. tan(A + B) = (1/2 + 1/3)/(1 - 1/2 · 1/3)  
   = (5/6)/(5/6) = **1**  
   (This means A + B = 45°!)

5. LHS = [sin A cos B + cos A sin B] + [sin A cos B - cos A sin B]  
   = 2 sin A cos B = RHS ✓

6. This comes from the geometric proof using the distance formula. The cosine of an angle difference is related to the dot product of unit vectors, which involves the pattern cos A cos B + sin A sin B for subtraction.

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 3.4 Proving Identities](../03-Trigonometric-Identities/04-proving-identities.md) | [Unit 4 Index](README.md) | [4.2 Product to Sum →](02-product-to-sum.md) |

---

[← Back to Main Index](../README.md)
