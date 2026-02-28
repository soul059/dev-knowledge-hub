# Chapter 4.4: Applications of Compound Angle Formulas

## Overview

This chapter consolidates all compound angle formulas and demonstrates their practical applications in solving problems, proving identities, and working with real-world scenarios involving waves, physics, and engineering.

---

## 📋 Complete Formula Reference

### Addition and Subtraction Formulas

| Formula Type | Expression |
|--------------|------------|
| sin(A + B) | sin A cos B + cos A sin B |
| sin(A - B) | sin A cos B - cos A sin B |
| cos(A + B) | cos A cos B - sin A sin B |
| cos(A - B) | cos A cos B + sin A sin B |
| tan(A + B) | (tan A + tan B)/(1 - tan A tan B) |
| tan(A - B) | (tan A - tan B)/(1 + tan A tan B) |

### Product to Sum Formulas

| Product | Sum/Difference |
|---------|----------------|
| 2 sin A cos B | sin(A+B) + sin(A-B) |
| 2 cos A sin B | sin(A+B) - sin(A-B) |
| 2 cos A cos B | cos(A+B) + cos(A-B) |
| 2 sin A sin B | cos(A-B) - cos(A+B) |

### Sum to Product Formulas

| Sum/Difference | Product |
|----------------|---------|
| sin A + sin B | 2 sin((A+B)/2) cos((A-B)/2) |
| sin A - sin B | 2 cos((A+B)/2) sin((A-B)/2) |
| cos A + cos B | 2 cos((A+B)/2) cos((A-B)/2) |
| cos A - cos B | -2 sin((A+B)/2) sin((A-B)/2) |

---

## 📐 Application Type 1: Finding Exact Values

### Standard Technique

```
    For angles like 15°, 75°, 105°, 165°, etc.
    
    Express as sum/difference of standard angles:
    ┌─────────────────────────────────────────┐
    │  15° = 45° - 30° = 60° - 45°           │
    │  75° = 45° + 30° = 120° - 45°          │
    │  105° = 60° + 45° = 180° - 75°         │
    │  165° = 180° - 15° = 120° + 45°        │
    └─────────────────────────────────────────┘
```

### Example 1: Find sin 15°

**Solution:**
$$\sin 15° = \sin(45° - 30°)$$
$$= \sin 45° \cos 30° - \cos 45° \sin 30°$$
$$= \frac{\sqrt{2}}{2} \cdot \frac{\sqrt{3}}{2} - \frac{\sqrt{2}}{2} \cdot \frac{1}{2}$$
$$= \frac{\sqrt{6}}{4} - \frac{\sqrt{2}}{4}$$
$$= \boxed{\frac{\sqrt{6} - \sqrt{2}}{4}}$$

### Example 2: Find tan 75°

**Solution:**
$$\tan 75° = \tan(45° + 30°)$$
$$= \frac{\tan 45° + \tan 30°}{1 - \tan 45° \tan 30°}$$
$$= \frac{1 + \frac{1}{\sqrt{3}}}{1 - 1 \cdot \frac{1}{\sqrt{3}}}$$
$$= \frac{\sqrt{3} + 1}{\sqrt{3} - 1}$$

Rationalizing:
$$= \frac{(\sqrt{3} + 1)^2}{(\sqrt{3} - 1)(\sqrt{3} + 1)}$$
$$= \frac{3 + 2\sqrt{3} + 1}{3 - 1}$$
$$= \frac{4 + 2\sqrt{3}}{2}$$
$$= \boxed{2 + \sqrt{3}}$$

### Example 3: Find cos 105°

**Solution:**
$$\cos 105° = \cos(60° + 45°)$$
$$= \cos 60° \cos 45° - \sin 60° \sin 45°$$
$$= \frac{1}{2} \cdot \frac{\sqrt{2}}{2} - \frac{\sqrt{3}}{2} \cdot \frac{\sqrt{2}}{2}$$
$$= \frac{\sqrt{2}}{4} - \frac{\sqrt{6}}{4}$$
$$= \boxed{\frac{\sqrt{2} - \sqrt{6}}{4}}$$

---

## 📐 Application Type 2: Proving Identities

### Strategy for Proving Identities

```
    ┌─────────────────────────────────────────────────────────┐
    │                  IDENTITY PROOF STRATEGY                │
    ├─────────────────────────────────────────────────────────┤
    │  1. Work with the more complex side                     │
    │  2. Apply compound angle formulas                       │
    │  3. Use sum/product transformations if needed           │
    │  4. Simplify using fundamental identities               │
    │  5. Arrive at the simpler side                         │
    └─────────────────────────────────────────────────────────┘
```

### Example 4: Prove sin(A+B) sin(A-B) = sin²A - sin²B

**Solution:**
$$\text{LHS} = \sin(A+B) \cdot \sin(A-B)$$
$$= (\sin A \cos B + \cos A \sin B)(\sin A \cos B - \cos A \sin B)$$

Using (x + y)(x - y) = x² - y²:
$$= \sin^2 A \cos^2 B - \cos^2 A \sin^2 B$$
$$= \sin^2 A (1 - \sin^2 B) - (1 - \sin^2 A) \sin^2 B$$
$$= \sin^2 A - \sin^2 A \sin^2 B - \sin^2 B + \sin^2 A \sin^2 B$$
$$= \sin^2 A - \sin^2 B = \text{RHS} \quad \checkmark$$

### Example 5: Prove cos(A+B) cos(A-B) = cos²A - sin²B

**Solution:**
$$\text{LHS} = \cos(A+B) \cdot \cos(A-B)$$
$$= (\cos A \cos B - \sin A \sin B)(\cos A \cos B + \sin A \sin B)$$
$$= \cos^2 A \cos^2 B - \sin^2 A \sin^2 B$$
$$= \cos^2 A (1 - \sin^2 B) - (1 - \cos^2 A) \sin^2 B$$
$$= \cos^2 A - \cos^2 A \sin^2 B - \sin^2 B + \cos^2 A \sin^2 B$$
$$= \cos^2 A - \sin^2 B = \text{RHS} \quad \checkmark$$

### Example 6: Prove tan(45° + A) · tan(45° - A) = 1

**Solution:**
$$\text{LHS} = \tan(45° + A) \cdot \tan(45° - A)$$
$$= \frac{1 + \tan A}{1 - \tan A} \cdot \frac{1 - \tan A}{1 + \tan A}$$
$$= \frac{(1 + \tan A)(1 - \tan A)}{(1 - \tan A)(1 + \tan A)}$$
$$= 1 = \text{RHS} \quad \checkmark$$

---

## 📐 Application Type 3: Solving Equations

### Example 7: Solve sin x + sin 3x = 0

**Solution:**
Using sum to product formula:
$$2\sin\left(\frac{x + 3x}{2}\right)\cos\left(\frac{x - 3x}{2}\right) = 0$$
$$2\sin 2x \cos(-x) = 0$$
$$2\sin 2x \cos x = 0$$

Either sin 2x = 0 or cos x = 0

**Case 1:** sin 2x = 0
$$2x = n\pi, \quad n \in \mathbb{Z}$$
$$x = \frac{n\pi}{2}$$

**Case 2:** cos x = 0
$$x = \frac{\pi}{2} + n\pi = \frac{(2n+1)\pi}{2}$$

Combining: $$\boxed{x = \frac{n\pi}{2}, \quad n \in \mathbb{Z}}$$

### Example 8: Solve cos 5x + cos 3x = 0

**Solution:**
Using sum to product:
$$2\cos\left(\frac{5x + 3x}{2}\right)\cos\left(\frac{5x - 3x}{2}\right) = 0$$
$$2\cos 4x \cos x = 0$$

Either cos 4x = 0 or cos x = 0

**Case 1:** cos 4x = 0
$$4x = \frac{\pi}{2} + n\pi$$
$$x = \frac{\pi}{8} + \frac{n\pi}{4} = \frac{(2n+1)\pi}{8}$$

**Case 2:** cos x = 0
$$x = \frac{(2n+1)\pi}{2}$$

---

## 📐 Application Type 4: Wave Combination

### Combining Sinusoidal Functions

When we have expressions of the form:
$$a\sin\theta + b\cos\theta$$

We can write it as:
$$R\sin(\theta + \phi)$$

where $R = \sqrt{a^2 + b^2}$ and $\tan\phi = \frac{b}{a}$

```
    ┌────────────────────────────────────────────────────────────┐
    │         CONVERSION: a sin θ + b cos θ = R sin(θ + φ)       │
    ├────────────────────────────────────────────────────────────┤
    │                                                            │
    │             R = √(a² + b²)                                 │
    │                                                            │
    │             tan φ = b/a                                    │
    │                                                            │
    │   Alternative: R cos(θ - ψ) where tan ψ = a/b             │
    │                                                            │
    └────────────────────────────────────────────────────────────┘
```

### Example 9: Express 3sin θ + 4cos θ in the form R sin(θ + φ)

**Solution:**
$$R = \sqrt{3^2 + 4^2} = \sqrt{9 + 16} = \sqrt{25} = 5$$

$$\tan\phi = \frac{4}{3}$$
$$\phi = \tan^{-1}\left(\frac{4}{3}\right) \approx 53.13°$$

$$\therefore 3\sin\theta + 4\cos\theta = 5\sin(\theta + 53.13°)$$

**Verification:**
$$5\sin(\theta + \phi) = 5(\sin\theta\cos\phi + \cos\theta\sin\phi)$$
$$= 5\sin\theta \cdot \frac{3}{5} + 5\cos\theta \cdot \frac{4}{5}$$
$$= 3\sin\theta + 4\cos\theta \quad \checkmark$$

### Example 10: Find maximum value of 5sin x + 12cos x

**Solution:**
$$R = \sqrt{5^2 + 12^2} = \sqrt{25 + 144} = \sqrt{169} = 13$$

Since the maximum value of sin(x + φ) is 1:

**Maximum value = 13**
**Minimum value = -13**

---

## 📐 Application Type 5: Physics Applications

### Simple Harmonic Motion

```
    Superposition of Two SHM Waves:
    
    y₁ = A sin(ωt)
    y₂ = A sin(ωt + φ)
    
    Combined: y = y₁ + y₂ = 2A cos(φ/2) sin(ωt + φ/2)
    
    ┌─────────────────────────────────────────────┐
    │     Resultant amplitude = 2A cos(φ/2)       │
    │     Phase shift = φ/2                       │
    └─────────────────────────────────────────────┘
```

### Example 11: Superposition of Waves

Two sound waves are given by y₁ = 5sin(100πt) and y₂ = 5sin(100πt + π/3).
Find the resultant wave.

**Solution:**
$$y = y_1 + y_2 = 5\sin(100\pi t) + 5\sin(100\pi t + \frac{\pi}{3})$$

Using sum to product with A = 100πt + π/3 and B = 100πt:
$$= 2 \cdot 5 \cdot \sin\left(\frac{(100\pi t + \frac{\pi}{3}) + 100\pi t}{2}\right)\cos\left(\frac{(100\pi t + \frac{\pi}{3}) - 100\pi t}{2}\right)$$
$$= 10\sin\left(100\pi t + \frac{\pi}{6}\right)\cos\left(\frac{\pi}{6}\right)$$
$$= 10 \cdot \frac{\sqrt{3}}{2} \cdot \sin\left(100\pi t + \frac{\pi}{6}\right)$$
$$= 5\sqrt{3}\sin\left(100\pi t + \frac{\pi}{6}\right)$$

**Resultant amplitude = 5√3 ≈ 8.66**

---

## 📊 Problem-Solving Flowchart

```
    ┌─────────────────────────────────────────────────────────────┐
    │                 COMPOUND ANGLE PROBLEM                      │
    └────────────────────────┬────────────────────────────────────┘
                             ↓
              ┌──────────────┴──────────────┐
              │   What type of problem?     │
              └──────────────┬──────────────┘
                             │
        ┌────────────┬───────┴───────┬────────────┐
        ↓            ↓               ↓            ↓
    ┌───────┐   ┌─────────┐   ┌──────────┐   ┌─────────┐
    │ Exact │   │ Prove   │   │  Solve   │   │Combine  │
    │ Value │   │Identity │   │ Equation │   │ Waves   │
    └───┬───┘   └────┬────┘   └────┬─────┘   └────┬────┘
        ↓            ↓             ↓              ↓
    Express as    Work with     Convert to     Find R and
    sum/diff of   complex       product        φ for
    standard      side first    form           R sin(θ+φ)
    angles
```

---

## 📐 Advanced Identities

### Triple Combination

$$\sin(A + B + C) = \sin A \cos B \cos C + \cos A \sin B \cos C$$
$$\quad\quad\quad\quad\quad\quad + \cos A \cos B \sin C - \sin A \sin B \sin C$$

Or equivalently:
$$\sin(A + B + C) = \sum\sin A \cos B \cos C - \sin A \sin B \sin C$$

### Tangent of Sum of Three Angles

$$\tan(A + B + C) = \frac{\tan A + \tan B + \tan C - \tan A \tan B \tan C}{1 - \tan A \tan B - \tan B \tan C - \tan C \tan A}$$

---

## 🎯 Important Special Results

### When A + B = 45°

$$\tan(A + B) = 1$$
$$\frac{\tan A + \tan B}{1 - \tan A \tan B} = 1$$
$$\tan A + \tan B = 1 - \tan A \tan B$$
$$\boxed{(1 + \tan A)(1 + \tan B) = 2}$$

### When A + B = 90°

$$\sin(A + B) = 1, \quad \cos(A + B) = 0$$
$$\sin A = \cos B, \quad \cos A = \sin B$$
$$\tan A \tan B = 1$$

### When A + B = 180°

$$\sin A = \sin B, \quad \cos A = -\cos B$$
$$\tan A = -\tan B$$

---

## 📋 Summary Table

### Application Types and Techniques

| Application | Technique | Key Formula(s) |
|-------------|-----------|----------------|
| Exact values | Express as sum/diff of standard angles | sin(A±B), cos(A±B), tan(A±B) |
| Proving identities | Expand using compound formulas | All compound angle formulas |
| Solving equations | Convert sum to product | Sum to product formulas |
| Combining waves | Express as R sin(θ + φ) | R = √(a² + b²), tan φ = b/a |
| Physics problems | Superposition of waves | All formulas as needed |

### Key Values to Remember

| Expression | Exact Value |
|------------|-------------|
| sin 15° | (√6 - √2)/4 |
| cos 15° | (√6 + √2)/4 |
| tan 15° | 2 - √3 |
| sin 75° | (√6 + √2)/4 |
| cos 75° | (√6 - √2)/4 |
| tan 75° | 2 + √3 |

---

## ❓ Quick Revision Questions

1. **Find the exact value of sin 105° + cos 105°.**

2. **Prove that: tan 3A - tan 2A - tan A = tan 3A tan 2A tan A**

3. **Express 5 sin θ - 12 cos θ in the form R sin(θ - φ). Find R and φ.**

4. **Solve: sin 5x - sin x = sin 3x for x ∈ [0, 2π]**

5. **If A + B = 45°, prove that (1 + tan A)(1 + tan B) = 2**

6. **Two waves y₁ = 3sin(ωt) and y₂ = 4cos(ωt) are superposed. Find the amplitude of the resultant wave.**

<details>
<summary>Click to see answers</summary>

1. sin 105° + cos 105°  
   = sin(60°+45°) + cos(60°+45°)  
   = (sin60°cos45° + cos60°sin45°) + (cos60°cos45° - sin60°sin45°)  
   = (√3/2)(√2/2) + (1/2)(√2/2) + (1/2)(√2/2) - (√3/2)(√2/2)  
   = √6/4 + √2/4 + √2/4 - √6/4  
   = **√2/2**

2. Since 3A = 2A + A:
   tan 3A = tan(2A + A) = (tan 2A + tan A)/(1 - tan 2A tan A)  
   tan 3A(1 - tan 2A tan A) = tan 2A + tan A  
   tan 3A - tan 3A tan 2A tan A = tan 2A + tan A  
   **tan 3A - tan 2A - tan A = tan 3A tan 2A tan A** ✓

3. R = √(5² + 12²) = √(25 + 144) = **13**  
   For R sin(θ - φ) = R(sin θ cos φ - cos θ sin φ)  
   Comparing: R cos φ = 5 and R sin φ = 12  
   tan φ = 12/5  
   **φ = tan⁻¹(12/5) ≈ 67.38°**

4. sin 5x - sin x = sin 3x  
   2 cos 3x sin 2x = sin 3x  
   2 cos 3x sin 2x - sin 3x = 0  
   sin 3x(2 cos 3x · (sin 2x/sin 3x) - 1) = 0  
   
   Actually, rearranging:  
   2 cos 3x sin 2x = sin 3x  
   If sin 3x ≠ 0: 2 cos 3x sin 2x = sin 3x  
   Using sin 3x = sin 2x cos x + cos 2x sin x... (complex)  
   
   Simpler: sin 5x - sin 3x = sin x  
   2 cos 4x sin x = sin x  
   sin x(2 cos 4x - 1) = 0  
   sin x = 0 → x = 0, π, 2π  
   cos 4x = 1/2 → 4x = π/3, 5π/3, 7π/3, 11π/3, 13π/3, 17π/3, 19π/3, 23π/3  
   x = **π/12, 5π/12, 7π/12, 11π/12, 13π/12, 17π/12, 19π/12, 23π/12, 0, π, 2π**

5. A + B = 45°, so tan(A + B) = 1  
   (tan A + tan B)/(1 - tan A tan B) = 1  
   tan A + tan B = 1 - tan A tan B  
   1 + tan A + tan B + tan A tan B = 2  
   (1 + tan A) + tan B(1 + tan A) = 2  
   **(1 + tan A)(1 + tan B) = 2** ✓

6. y = 3 sin(ωt) + 4 cos(ωt) = 3 sin(ωt) + 4 sin(ωt + π/2)  
   R = √(3² + 4²) = √25 = **5**  
   The resultant amplitude is **5**.

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 4.3 Sum to Product](03-sum-to-product.md) | [Unit 4 Index](README.md) | [Unit 5: Multiple Angles →](../05-Multiple-Submultiple-Angles/README.md) |

---

[← Back to Main Index](../README.md)
