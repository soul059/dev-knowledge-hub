# Chapter 4.2: Product to Sum Formulas

## Overview

Product to sum formulas allow us to convert **products** of trigonometric functions into **sums or differences**. These formulas are derived from the compound angle formulas and are useful for integration, simplifying expressions, and signal analysis.

---

## 📐 The Product to Sum Formulas

### The Four Formulas

$$\boxed{\sin A \cos B = \frac{1}{2}[\sin(A + B) + \sin(A - B)]}$$

$$\boxed{\cos A \sin B = \frac{1}{2}[\sin(A + B) - \sin(A - B)]}$$

$$\boxed{\cos A \cos B = \frac{1}{2}[\cos(A + B) + \cos(A - B)]}$$

$$\boxed{\sin A \sin B = \frac{1}{2}[\cos(A - B) - \cos(A + B)]}$$

---

## 🔍 Derivation of Product to Sum Formulas

### Deriving sin A cos B Formula

Start with the addition and subtraction formulas:
$$\sin(A + B) = \sin A \cos B + \cos A \sin B \quad \text{...(1)}$$
$$\sin(A - B) = \sin A \cos B - \cos A \sin B \quad \text{...(2)}$$

**Add equations (1) and (2):**
$$\sin(A + B) + \sin(A - B) = 2\sin A \cos B$$

$$\therefore \sin A \cos B = \frac{1}{2}[\sin(A + B) + \sin(A - B)]$$

### Deriving cos A sin B Formula

**Subtract equation (2) from (1):**
$$\sin(A + B) - \sin(A - B) = 2\cos A \sin B$$

$$\therefore \cos A \sin B = \frac{1}{2}[\sin(A + B) - \sin(A - B)]$$

### Deriving cos A cos B Formula

Start with:
$$\cos(A + B) = \cos A \cos B - \sin A \sin B \quad \text{...(3)}$$
$$\cos(A - B) = \cos A \cos B + \sin A \sin B \quad \text{...(4)}$$

**Add equations (3) and (4):**
$$\cos(A + B) + \cos(A - B) = 2\cos A \cos B$$

$$\therefore \cos A \cos B = \frac{1}{2}[\cos(A + B) + \cos(A - B)]$$

### Deriving sin A sin B Formula

**Subtract equation (3) from (4):**
$$\cos(A - B) - \cos(A + B) = 2\sin A \sin B$$

$$\therefore \sin A \sin B = \frac{1}{2}[\cos(A - B) - \cos(A + B)]$$

---

## 📊 Formula Reference Chart

```
    ┌───────────────────────────────────────────────────────────────┐
    │              PRODUCT TO SUM FORMULAS                          │
    ├───────────────────────────────────────────────────────────────┤
    │                                                               │
    │  sin A cos B = ½[sin(A+B) + sin(A-B)]     → "sin + sin"      │
    │                                                               │
    │  cos A sin B = ½[sin(A+B) - sin(A-B)]     → "sin - sin"      │
    │                                                               │
    │  cos A cos B = ½[cos(A+B) + cos(A-B)]     → "cos + cos"      │
    │                                                               │
    │  sin A sin B = ½[cos(A-B) - cos(A+B)]     → "cos - cos"      │
    │                          ↑       ↑                            │
    │                      NOTE: Order reversed!                    │
    │                                                               │
    └───────────────────────────────────────────────────────────────┘
```

---

## 🧠 Memory Techniques

### Pattern Recognition

```
    Product        →    Sum
    ─────────           ───────
    sin · cos     →    sin ± sin
    cos · sin     →    sin ± sin
    cos · cos     →    cos + cos
    sin · sin     →    cos - cos (reversed order!)
```

### The "SCS Rule"

- **S**in × **C**os = **S**in terms
- **C**os × **C**os = **C**os terms (with +)
- **S**in × **S**in = **C**os terms (with -, reversed)

---

## 🧮 Worked Examples

### Example 1: Basic Conversion

Express sin 5x cos 3x as a sum.

**Solution:**
Using sin A cos B = ½[sin(A+B) + sin(A-B)]:
$$\sin 5x \cos 3x = \frac{1}{2}[\sin(5x + 3x) + \sin(5x - 3x)]$$
$$= \frac{1}{2}[\sin 8x + \sin 2x]$$
$$= \boxed{\frac{1}{2}\sin 8x + \frac{1}{2}\sin 2x}$$

### Example 2: cos × cos Product

Express cos 4θ cos 2θ as a sum.

**Solution:**
Using cos A cos B = ½[cos(A+B) + cos(A-B)]:
$$\cos 4\theta \cos 2\theta = \frac{1}{2}[\cos(4\theta + 2\theta) + \cos(4\theta - 2\theta)]$$
$$= \frac{1}{2}[\cos 6\theta + \cos 2\theta]$$
$$= \boxed{\frac{1}{2}\cos 6\theta + \frac{1}{2}\cos 2\theta}$$

### Example 3: sin × sin Product

Express sin 3x sin x as a sum.

**Solution:**
Using sin A sin B = ½[cos(A-B) - cos(A+B)]:
$$\sin 3x \sin x = \frac{1}{2}[\cos(3x - x) - \cos(3x + x)]$$
$$= \frac{1}{2}[\cos 2x - \cos 4x]$$
$$= \boxed{\frac{1}{2}\cos 2x - \frac{1}{2}\cos 4x}$$

### Example 4: Numerical Values

Find the exact value of sin 75° cos 15°.

**Solution:**
$$\sin 75° \cos 15° = \frac{1}{2}[\sin(75° + 15°) + \sin(75° - 15°)]$$
$$= \frac{1}{2}[\sin 90° + \sin 60°]$$
$$= \frac{1}{2}\left[1 + \frac{\sqrt{3}}{2}\right]$$
$$= \frac{1}{2} + \frac{\sqrt{3}}{4}$$
$$= \boxed{\frac{2 + \sqrt{3}}{4}}$$

### Example 5: cos × sin Product

Express cos 7x sin 3x as a sum.

**Solution:**
Using cos A sin B = ½[sin(A+B) - sin(A-B)]:
$$\cos 7x \sin 3x = \frac{1}{2}[\sin(7x + 3x) - \sin(7x - 3x)]$$
$$= \frac{1}{2}[\sin 10x - \sin 4x]$$
$$= \boxed{\frac{1}{2}\sin 10x - \frac{1}{2}\sin 4x}$$

### Example 6: Simplifying Complex Products

Simplify: 2 sin 50° cos 10°

**Solution:**
$$2 \sin 50° \cos 10° = 2 \cdot \frac{1}{2}[\sin(50° + 10°) + \sin(50° - 10°)]$$
$$= \sin 60° + \sin 40°$$
$$= \boxed{\frac{\sqrt{3}}{2} + \sin 40°}$$

---

## 📐 Special Cases

### When A = B

$$\sin A \cos A = \frac{1}{2}[\sin 2A + \sin 0] = \frac{1}{2}\sin 2A$$

$$\cos A \cos A = \frac{1}{2}[\cos 2A + \cos 0] = \frac{1}{2}[\cos 2A + 1] = \frac{1 + \cos 2A}{2}$$

$$\sin A \sin A = \frac{1}{2}[\cos 0 - \cos 2A] = \frac{1}{2}[1 - \cos 2A] = \frac{1 - \cos 2A}{2}$$

These lead to the **power reduction formulas** (covered in Unit 5).

---

## 📊 Comparison Table

| Product Form | Sum Form |
|--------------|----------|
| sin A cos B | ½[sin(A+B) + sin(A-B)] |
| cos A sin B | ½[sin(A+B) - sin(A-B)] |
| cos A cos B | ½[cos(A+B) + cos(A-B)] |
| sin A sin B | ½[cos(A-B) - cos(A+B)] |

---

## 🎯 Applications in Integration

### Why These Formulas Matter

Products of trigonometric functions are difficult to integrate directly. Converting to sums makes integration straightforward.

```
    ∫ sin mx cos nx dx = ∫ ½[sin(m+n)x + sin(m-n)x] dx
    
    This is much easier to integrate!
```

**Example:**
$$\int \sin 3x \cos 2x \, dx = \int \frac{1}{2}[\sin 5x + \sin x] \, dx$$
$$= \frac{1}{2}\left[-\frac{\cos 5x}{5} - \cos x\right] + C$$
$$= -\frac{\cos 5x}{10} - \frac{\cos x}{2} + C$$

---

## 📈 Visualization

### Product vs Sum Representation

```
    Product: sin A cos B
    
    A wave (sin A) modulated by another wave (cos B)
    
        ~~~∿~~~∿~~~∿~~~∿~~~   × 
        ___/‾‾‾\___/‾‾‾\___  
        ========================
        Complex combined pattern
    
    Sum: ½[sin(A+B) + sin(A-B)]
    
        ~~~∿~~~∿~~~∿~~~∿~~~   +
        __∿__∿__∿__∿__∿__  
        ========================
        Sum of two simple waves
```

---

## 🌍 Real-World Applications

### 1. Signal Processing
Radio signals use product-to-sum conversions for modulation/demodulation.

### 2. Acoustics
Beat frequencies occur when two sound waves of slightly different frequencies combine.

### 3. Music Theory
Harmonic analysis of musical chords uses these formulas.

### 4. Electronics
Mixers in radio receivers multiply signals, then use these identities.

---

## 📋 Summary Table

### Quick Reference

| If You Have | Use This Formula |
|-------------|------------------|
| sin × cos | ½[sin(sum) + sin(diff)] |
| cos × sin | ½[sin(sum) - sin(diff)] |
| cos × cos | ½[cos(sum) + cos(diff)] |
| sin × sin | ½[cos(diff) - cos(sum)] |

### Key Points to Remember

| Point | Detail |
|-------|--------|
| Factor of ½ | Always present in result |
| sin × cos → sin | Product with different functions gives sin |
| cos × cos → cos | Same functions (cos) give cos |
| sin × sin → cos | Same functions (sin) give cos (reversed!) |
| Order matters for sin × sin | It's cos(A-B) - cos(A+B), not the other way |

---

## ❓ Quick Revision Questions

1. **Express sin 6x cos 2x as a sum.**

2. **Express cos 5θ cos 3θ as a sum.**

3. **Express sin 4x sin 2x as a sum.**

4. **Find the exact value of cos 75° cos 15°.**

5. **Prove that sin 3A cos A + cos 3A sin A = sin 4A using product formulas.**

6. **Why is the sin A sin B formula different from the others (reversed order)?**

<details>
<summary>Click to see answers</summary>

1. sin 6x cos 2x = ½[sin 8x + sin 4x]

2. cos 5θ cos 3θ = ½[cos 8θ + cos 2θ]

3. sin 4x sin 2x = ½[cos 2x - cos 6x]

4. cos 75° cos 15° = ½[cos 90° + cos 60°]  
   = ½[0 + ½] = **1/4**

5. LHS = sin 3A cos A + cos 3A sin A  
   = sin(3A + A) [using addition formula]  
   = sin 4A = RHS ✓  
   (Note: This is actually the addition formula directly!)

6. When deriving from cos(A-B) - cos(A+B) = 2 sin A sin B, the subtraction of cosines naturally gives cos(A-B) first. The negative sign attaches to cos(A+B), resulting in the reversed order.

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 4.1 Addition Formulas](01-addition-formulas.md) | [Unit 4 Index](README.md) | [4.3 Sum to Product →](03-sum-to-product.md) |

---

[← Back to Main Index](../README.md)
