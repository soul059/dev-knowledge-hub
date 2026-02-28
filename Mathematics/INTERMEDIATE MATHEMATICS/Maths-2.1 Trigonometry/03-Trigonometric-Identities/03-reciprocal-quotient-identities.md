# Chapter 3.3: Reciprocal and Quotient Identities

## Overview

Reciprocal and quotient identities express relationships between the six trigonometric functions. These identities are fundamental for converting between different trigonometric forms and simplifying complex expressions.

---

## 📐 Reciprocal Identities

### Definition

Reciprocal identities relate each trigonometric function to its reciprocal counterpart:

$$\csc\theta = \frac{1}{\sin\theta} \qquad \sin\theta = \frac{1}{\csc\theta}$$

$$\sec\theta = \frac{1}{\cos\theta} \qquad \cos\theta = \frac{1}{\sec\theta}$$

$$\cot\theta = \frac{1}{\tan\theta} \qquad \tan\theta = \frac{1}{\cot\theta}$$

### Product Form

When a function is multiplied by its reciprocal, the result is always 1:

$$\sin\theta \cdot \csc\theta = 1$$
$$\cos\theta \cdot \sec\theta = 1$$
$$\tan\theta \cdot \cot\theta = 1$$

---

## 🔗 Relationship Diagram

```
    ┌───────────────────────────────────────────────────────────┐
    │               RECIPROCAL PAIRS                            │
    ├───────────────────────────────────────────────────────────┤
    │                                                           │
    │      sin θ  ←─────── reciprocal ───────→  csc θ          │
    │         │                                     │           │
    │         │                                     │           │
    │      cos θ  ←─────── reciprocal ───────→  sec θ          │
    │         │                                     │           │
    │         │                                     │           │
    │      tan θ  ←─────── reciprocal ───────→  cot θ          │
    │                                                           │
    │   Memory: sin↔csc, cos↔sec, tan↔cot                      │
    │                                                           │
    └───────────────────────────────────────────────────────────┘
```

### Memory Aid

```
    The "co-" prefix helps pair functions:
    
    sin  ←→  csc (co-secant)
    ↓         ↓
    co-sine   secant
    cos  ←→  sec
    
    tan  ←→  cot (co-tangent)
    
    Note: Functions WITHOUT "co-" pair with functions WITH "co-"
          (Exception: the sin-csc pair)
```

---

## 📊 Quotient Identities

### Definition

Quotient identities express tangent and cotangent as ratios of sine and cosine:

$$\tan\theta = \frac{\sin\theta}{\cos\theta}$$

$$\cot\theta = \frac{\cos\theta}{\sin\theta}$$

### Derivation from Right Triangle

```
              |\
              | \
    Opposite  |  \ Hypotenuse
         (O)  |   \
              |    \
              |  θ  \
              |______\
              Adjacent (A)
              
    tan θ = O/A = (O/H)/(A/H) = sin θ / cos θ
    
    cot θ = A/O = (A/H)/(O/H) = cos θ / sin θ
```

### Relationship

$$\tan\theta \cdot \cot\theta = \frac{\sin\theta}{\cos\theta} \cdot \frac{\cos\theta}{\sin\theta} = 1$$

This confirms that tan and cot are reciprocals.

---

## 📈 Complete Relationship Map

```
                              1
                             /|\
                            / | \
                           /  |  \
                          /   |   \
                    csc θ    sec θ    cot θ
                      │        │        │
                reciprocal  reciprocal  reciprocal
                      │        │        │
                    sin θ    cos θ    tan θ
                      \       / \       /
                       \     /   \     /
                        \   /     \   /
                      quotient   quotient
                         ↓          ↓
                       tan θ     cot θ
                       
    tan θ = sin θ / cos θ
    cot θ = cos θ / sin θ
```

---

## 🧮 Worked Examples

### Example 1: Converting to Sine and Cosine

Express sec θ + tan θ in terms of sin θ and cos θ.

**Solution:**
$$\sec\theta + \tan\theta = \frac{1}{\cos\theta} + \frac{\sin\theta}{\cos\theta}$$
$$= \frac{1 + \sin\theta}{\cos\theta}$$

### Example 2: Simplifying Using Reciprocals

Simplify: $\frac{\sin\theta \cdot \sec\theta}{\tan\theta}$

**Solution:**
$$= \frac{\sin\theta \cdot \frac{1}{\cos\theta}}{\frac{\sin\theta}{\cos\theta}}$$
$$= \frac{\frac{\sin\theta}{\cos\theta}}{\frac{\sin\theta}{\cos\theta}}$$
$$= \boxed{1}$$

**Alternative method:**
$$= \frac{\sin\theta \cdot \sec\theta}{\tan\theta} = \frac{\sin\theta}{\cos\theta} \cdot \frac{1}{\tan\theta}$$
$$= \tan\theta \cdot \cot\theta = 1$$

### Example 3: Proving an Identity

Prove: $\cot\theta + \tan\theta = \csc\theta \cdot \sec\theta$

**Proof:**
$$LHS = \cot\theta + \tan\theta$$
$$= \frac{\cos\theta}{\sin\theta} + \frac{\sin\theta}{\cos\theta}$$
$$= \frac{\cos^2\theta + \sin^2\theta}{\sin\theta \cdot \cos\theta}$$
$$= \frac{1}{\sin\theta \cdot \cos\theta}$$
$$= \frac{1}{\sin\theta} \cdot \frac{1}{\cos\theta}$$
$$= \csc\theta \cdot \sec\theta = RHS \quad \checkmark$$

### Example 4: Finding Values

If sin θ = 3/5 and θ is acute, find all six trigonometric values.

**Solution:**

Step 1: Find cos θ using Pythagorean identity
$$\cos^2\theta = 1 - \sin^2\theta = 1 - \frac{9}{25} = \frac{16}{25}$$
$$\cos\theta = \frac{4}{5}$$ (positive since θ is acute)

Step 2: Use quotient identities
$$\tan\theta = \frac{\sin\theta}{\cos\theta} = \frac{3/5}{4/5} = \frac{3}{4}$$
$$\cot\theta = \frac{\cos\theta}{\sin\theta} = \frac{4/5}{3/5} = \frac{4}{3}$$

Step 3: Use reciprocal identities
$$\csc\theta = \frac{1}{\sin\theta} = \frac{5}{3}$$
$$\sec\theta = \frac{1}{\cos\theta} = \frac{5}{4}$$

| Function | Value |
|----------|-------|
| sin θ | 3/5 |
| cos θ | 4/5 |
| tan θ | 3/4 |
| cot θ | 4/3 |
| sec θ | 5/4 |
| csc θ | 5/3 |

### Example 5: Complex Simplification

Simplify: $\frac{\sec^2\theta - 1}{\sec^2\theta}$

**Solution:**
$$= \frac{\sec^2\theta}{\sec^2\theta} - \frac{1}{\sec^2\theta}$$
$$= 1 - \cos^2\theta$$
$$= \sin^2\theta$$

**Alternative using identity sec²θ - 1 = tan²θ:**
$$= \frac{\tan^2\theta}{\sec^2\theta} = \tan^2\theta \cdot \cos^2\theta$$
$$= \frac{\sin^2\theta}{\cos^2\theta} \cdot \cos^2\theta = \sin^2\theta$$

---

## 📝 Extended Quotient Relationships

### All Functions in Terms of Sine and Cosine

| Function | In Terms of sin θ and cos θ |
|----------|----------------------------|
| sin θ | sin θ |
| cos θ | cos θ |
| tan θ | sin θ / cos θ |
| cot θ | cos θ / sin θ |
| sec θ | 1 / cos θ |
| csc θ | 1 / sin θ |

### Additional Quotient Forms

$$\tan\theta = \frac{\sec\theta}{\csc\theta} = \frac{1/\cos\theta}{1/\sin\theta} = \frac{\sin\theta}{\cos\theta}$$

$$\cot\theta = \frac{\csc\theta}{\sec\theta} = \frac{1/\sin\theta}{1/\cos\theta} = \frac{\cos\theta}{\sin\theta}$$

---

## 🔄 Converting Between Forms

### Strategy Guide

```
    ┌──────────────────────────────────────────────────────────┐
    │         CONVERSION STRATEGY                              │
    ├──────────────────────────────────────────────────────────┤
    │                                                          │
    │  To simplify complex expressions:                        │
    │                                                          │
    │  1. Convert all functions to sin and cos                 │
    │     • sec θ → 1/cos θ                                    │
    │     • csc θ → 1/sin θ                                    │
    │     • tan θ → sin θ/cos θ                                │
    │     • cot θ → cos θ/sin θ                                │
    │                                                          │
    │  2. Find common denominators                             │
    │                                                          │
    │  3. Simplify using Pythagorean identity                  │
    │                                                          │
    │  4. Convert back to other functions if needed            │
    │                                                          │
    └──────────────────────────────────────────────────────────┘
```

### Example: Multi-Step Conversion

Simplify: $\frac{\csc\theta - \cot\theta}{\sec\theta - 1}$

**Solution:**

Step 1: Convert to sin and cos
$$= \frac{\frac{1}{\sin\theta} - \frac{\cos\theta}{\sin\theta}}{\frac{1}{\cos\theta} - 1}$$

Step 2: Combine fractions
$$= \frac{\frac{1 - \cos\theta}{\sin\theta}}{\frac{1 - \cos\theta}{\cos\theta}}$$

Step 3: Divide fractions
$$= \frac{1 - \cos\theta}{\sin\theta} \cdot \frac{\cos\theta}{1 - \cos\theta}$$

Step 4: Cancel common terms
$$= \frac{\cos\theta}{\sin\theta} = \boxed{\cot\theta}$$

---

## 📊 Domain Restrictions

### When Functions Are Undefined

| Function | Undefined When |
|----------|---------------|
| tan θ = sin θ/cos θ | cos θ = 0 (θ = 90°, 270°, ...) |
| cot θ = cos θ/sin θ | sin θ = 0 (θ = 0°, 180°, ...) |
| sec θ = 1/cos θ | cos θ = 0 (θ = 90°, 270°, ...) |
| csc θ = 1/sin θ | sin θ = 0 (θ = 0°, 180°, ...) |

```
    Where tan θ and sec θ are undefined:
    
          │              │              │
          │   defined    │   defined    │
          │              │              │
    ──────┼──────────────┼──────────────┼──────
         90°           270°           450°
          ↑              ↑              ↑
      undefined      undefined      undefined
      (cos θ = 0)    (cos θ = 0)   (cos θ = 0)
```

---

## 🧮 Practice Problems Walkthrough

### Problem 1: Verify Identity

Verify: $\tan\theta + \cot\theta = \sec\theta \csc\theta$

**Verification:**
$$LHS = \tan\theta + \cot\theta$$

Convert to sin/cos:
$$= \frac{\sin\theta}{\cos\theta} + \frac{\cos\theta}{\sin\theta}$$

Common denominator:
$$= \frac{\sin^2\theta + \cos^2\theta}{\sin\theta \cos\theta}$$

Apply Pythagorean identity:
$$= \frac{1}{\sin\theta \cos\theta}$$

$$= \frac{1}{\sin\theta} \cdot \frac{1}{\cos\theta}$$

$$= \csc\theta \cdot \sec\theta = RHS \quad \checkmark$$

### Problem 2: Simplify

Simplify: $(\sin\theta + \cos\theta)^2 + (\sin\theta - \cos\theta)^2$

**Solution:**
$$= \sin^2\theta + 2\sin\theta\cos\theta + \cos^2\theta + \sin^2\theta - 2\sin\theta\cos\theta + \cos^2\theta$$
$$= 2\sin^2\theta + 2\cos^2\theta$$
$$= 2(\sin^2\theta + \cos^2\theta)$$
$$= 2(1) = \boxed{2}$$

---

## 🌍 Real-World Applications

### 1. Optics
Snell's law uses ratios of trigonometric functions to describe light refraction.

### 2. Mechanics
Force component calculations use quotient relationships between sin and cos.

### 3. Surveying
Slope calculations involve tan = sin/cos for grade measurements.

### 4. Electronics
Impedance calculations in AC circuits use reciprocal relationships.

---

## 📋 Summary Table

### Reciprocal Identities

| Pair | Relationship | Product |
|------|--------------|---------|
| sin θ, csc θ | csc θ = 1/sin θ | sin θ · csc θ = 1 |
| cos θ, sec θ | sec θ = 1/cos θ | cos θ · sec θ = 1 |
| tan θ, cot θ | cot θ = 1/tan θ | tan θ · cot θ = 1 |

### Quotient Identities

| Quotient | Definition | Domain Restriction |
|----------|------------|-------------------|
| tan θ | sin θ / cos θ | cos θ ≠ 0 |
| cot θ | cos θ / sin θ | sin θ ≠ 0 |

### Quick Conversion Reference

| From | To sin/cos |
|------|------------|
| tan θ | sin θ / cos θ |
| cot θ | cos θ / sin θ |
| sec θ | 1 / cos θ |
| csc θ | 1 / sin θ |

---

## ❓ Quick Revision Questions

1. **Write cot θ in terms of sin θ and cos θ.**

2. **Simplify: $\sin\theta \cdot \csc\theta \cdot \tan\theta \cdot \cot\theta$**

3. **If tan θ = 2 and θ is in Quadrant I, find sin θ and cos θ.**

4. **Prove: $\frac{1 - \sin\theta}{\cos\theta} = \frac{\cos\theta}{1 + \sin\theta}$**

5. **Express $\frac{\tan\theta}{\sec\theta}$ in simplest form.**

6. **Why is csc 0° undefined?**

<details>
<summary>Click to see answers</summary>

1. $\cot\theta = \frac{\cos\theta}{\sin\theta}$

2. $= (sin\theta \cdot \csc\theta) \cdot (tan\theta \cdot \cot\theta) = 1 \cdot 1 = \boxed{1}$

3. Since tan²θ + 1 = sec²θ:  
   sec²θ = 4 + 1 = 5, so sec θ = √5 (positive in Q I)  
   cos θ = 1/√5 = **√5/5**  
   sin θ = tan θ · cos θ = 2 · (√5/5) = **2√5/5**

4. Cross multiply: $(1 - \sin\theta)(1 + \sin\theta) = \cos^2\theta$  
   LHS = $1 - \sin^2\theta = \cos^2\theta$ = RHS ✓

5. $\frac{\tan\theta}{\sec\theta} = \frac{\sin\theta/\cos\theta}{1/\cos\theta} = \sin\theta \cdot \cos\theta \cdot \frac{1}{\cos\theta} = \boxed{\sin\theta}$

6. csc 0° = 1/sin 0° = 1/0, which is **undefined** (division by zero).

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 3.2 Pythagorean Identities](02-pythagorean-identities.md) | [Unit 3 Index](README.md) | [3.4 Proving Identities →](04-proving-identities.md) |

---

[← Back to Main Index](../README.md)
