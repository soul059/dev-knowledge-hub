# Chapter 3.1: Fundamental Identities

## Overview

Fundamental identities are the core relationships between trigonometric functions that hold true for all valid angle values. These identities form the foundation for all trigonometric simplifications and proofs. Mastering them is essential for success in trigonometry.

---

## 📐 Categories of Fundamental Identities

There are three main categories of fundamental identities:

```
    ┌─────────────────────────────────────────────────────────────┐
    │              FUNDAMENTAL TRIGONOMETRIC IDENTITIES           │
    ├─────────────────────────────────────────────────────────────┤
    │                                                             │
    │   1. RECIPROCAL IDENTITIES                                  │
    │      Relate each function to its reciprocal                 │
    │                                                             │
    │   2. QUOTIENT IDENTITIES                                    │
    │      Express tan and cot in terms of sin and cos            │
    │                                                             │
    │   3. PYTHAGOREAN IDENTITIES                                 │
    │      Based on the equation sin²θ + cos²θ = 1                │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

---

## 📊 Complete List of Fundamental Identities

### 1. Reciprocal Identities

$$\csc \theta = \frac{1}{\sin \theta}$$

$$\sec \theta = \frac{1}{\cos \theta}$$

$$\cot \theta = \frac{1}{\tan \theta}$$

**In product form:**

$$\sin \theta \cdot \csc \theta = 1$$
$$\cos \theta \cdot \sec \theta = 1$$
$$\tan \theta \cdot \cot \theta = 1$$

### 2. Quotient Identities

$$\tan \theta = \frac{\sin \theta}{\cos \theta}$$

$$\cot \theta = \frac{\cos \theta}{\sin \theta}$$

### 3. Pythagorean Identities

$$\sin^2 \theta + \cos^2 \theta = 1$$

$$1 + \tan^2 \theta = \sec^2 \theta$$

$$1 + \cot^2 \theta = \csc^2 \theta$$

---

## 🔗 Relationship Diagram

```
                              1
                              │
           ┌──────────────────┼──────────────────┐
           │                  │                  │
           ▼                  ▼                  ▼
         csc θ              sec θ              cot θ
           │                  │                  │
           │ reciprocal       │ reciprocal       │ reciprocal
           │                  │                  │
           ▼                  ▼                  ▼
         sin θ ──────────── cos θ ──────────── tan θ
                   │                  │
                   │    quotient      │
                   └──────────────────┘
                        tan θ = sin θ / cos θ
                        
    Connection via Pythagorean Identity:
    sin²θ + cos²θ = 1
```

---

## 📐 Deriving the Pythagorean Identities

### First Identity: sin²θ + cos²θ = 1

**From the Unit Circle:**

On the unit circle, any point is (cos θ, sin θ), and since x² + y² = 1:

$$\cos^2 \theta + \sin^2 \theta = 1$$

**From Right Triangle:**

```
              |\
              | \
            b |  \ c
              |   \
              |θ   \
              |_____\
                 a
                 
    sin θ = b/c,  cos θ = a/c
    
    sin²θ + cos²θ = (b/c)² + (a/c)²
                  = (b² + a²)/c²
                  = c²/c²        [Pythagorean theorem]
                  = 1 ✓
```

### Second Identity: 1 + tan²θ = sec²θ

**Derivation:**

Start with sin²θ + cos²θ = 1

Divide everything by cos²θ:

$$\frac{\sin^2 \theta}{\cos^2 \theta} + \frac{\cos^2 \theta}{\cos^2 \theta} = \frac{1}{\cos^2 \theta}$$

$$\tan^2 \theta + 1 = \sec^2 \theta$$

### Third Identity: 1 + cot²θ = csc²θ

**Derivation:**

Start with sin²θ + cos²θ = 1

Divide everything by sin²θ:

$$\frac{\sin^2 \theta}{\sin^2 \theta} + \frac{\cos^2 \theta}{\sin^2 \theta} = \frac{1}{\sin^2 \theta}$$

$$1 + \cot^2 \theta = \csc^2 \theta$$

---

## 📊 Alternative Forms of Pythagorean Identities

Each Pythagorean identity can be rearranged:

### From sin²θ + cos²θ = 1

| Form | Useful When |
|------|-------------|
| sin²θ = 1 - cos²θ | Eliminating sin² |
| cos²θ = 1 - sin²θ | Eliminating cos² |
| sin θ = ±√(1 - cos²θ) | Finding sin from cos |
| cos θ = ±√(1 - sin²θ) | Finding cos from sin |

### From 1 + tan²θ = sec²θ

| Form | Useful When |
|------|-------------|
| tan²θ = sec²θ - 1 | Eliminating tan² |
| sec²θ - tan²θ = 1 | Factoring |
| sec θ = ±√(1 + tan²θ) | Finding sec from tan |

### From 1 + cot²θ = csc²θ

| Form | Useful When |
|------|-------------|
| cot²θ = csc²θ - 1 | Eliminating cot² |
| csc²θ - cot²θ = 1 | Factoring |
| csc θ = ±√(1 + cot²θ) | Finding csc from cot |

---

## 🧮 Worked Examples

### Example 1: Simplify Using Reciprocal Identity

Simplify: $\sin \theta \cdot \csc \theta + \cos \theta \cdot \sec \theta$

**Solution:**
$$= (\sin \theta \cdot \csc \theta) + (\cos \theta \cdot \sec \theta)$$
$$= 1 + 1$$
$$= \boxed{2}$$

### Example 2: Convert to Sine and Cosine

Express tan θ + cot θ in terms of sin θ and cos θ, then simplify.

**Solution:**
$$\tan \theta + \cot \theta = \frac{\sin \theta}{\cos \theta} + \frac{\cos \theta}{\sin \theta}$$

$$= \frac{\sin^2 \theta + \cos^2 \theta}{\sin \theta \cos \theta}$$

$$= \frac{1}{\sin \theta \cos \theta}$$

$$= \boxed{\csc \theta \sec \theta}$$

### Example 3: Using Pythagorean Identity

If sin θ = 3/5 and θ is in Quadrant I, find cos θ.

**Solution:**
Using sin²θ + cos²θ = 1:
$$\left(\frac{3}{5}\right)^2 + \cos^2 \theta = 1$$
$$\frac{9}{25} + \cos^2 \theta = 1$$
$$\cos^2 \theta = 1 - \frac{9}{25} = \frac{16}{25}$$
$$\cos \theta = \pm \frac{4}{5}$$

Since θ is in Quadrant I (where cos is positive):
$$\cos \theta = \boxed{\frac{4}{5}}$$

### Example 4: Verify an Identity

Verify: $\sec^2 \theta - \tan^2 \theta = 1$

**Verification:**
$$\sec^2 \theta - \tan^2 \theta = (1 + \tan^2 \theta) - \tan^2 \theta$$
$$= 1 + \tan^2 \theta - \tan^2 \theta$$
$$= 1 \checkmark$$

### Example 5: Finding All Ratios

If tan θ = 2 and θ is in Quadrant III, find all six trigonometric ratios.

**Solution:**

Step 1: Use 1 + tan²θ = sec²θ
$$1 + 4 = \sec^2 \theta$$
$$\sec^2 \theta = 5$$
$$\sec \theta = \pm\sqrt{5}$$

In Q III, sec θ is negative: $\sec \theta = -\sqrt{5}$

Step 2: Find cos θ
$$\cos \theta = \frac{1}{\sec \theta} = -\frac{1}{\sqrt{5}} = -\frac{\sqrt{5}}{5}$$

Step 3: Find sin θ using tan θ = sin θ/cos θ
$$\sin \theta = \tan \theta \cdot \cos \theta = 2 \cdot \left(-\frac{\sqrt{5}}{5}\right) = -\frac{2\sqrt{5}}{5}$$

Step 4: Find remaining ratios

| Ratio | Value |
|-------|-------|
| sin θ | -2√5/5 |
| cos θ | -√5/5 |
| tan θ | 2 |
| csc θ | -√5/2 |
| sec θ | -√5 |
| cot θ | 1/2 |

---

## 📝 Co-function Identities

For complementary angles (θ and 90° - θ):

$$\sin(90° - \theta) = \cos \theta$$
$$\cos(90° - \theta) = \sin \theta$$
$$\tan(90° - \theta) = \cot \theta$$
$$\cot(90° - \theta) = \tan \theta$$
$$\sec(90° - \theta) = \csc \theta$$
$$\csc(90° - \theta) = \sec \theta$$

### Why "Co-function"?

The prefix "co-" in cosine, cotangent, and cosecant comes from "complementary."

```
    co-sine     = sine of complement
    co-tangent  = tangent of complement
    co-secant   = secant of complement
```

---

## 🔄 Even-Odd Identities

| Function | Type | Identity |
|----------|------|----------|
| sin(-θ) | Odd | -sin θ |
| cos(-θ) | Even | cos θ |
| tan(-θ) | Odd | -tan θ |
| cot(-θ) | Odd | -cot θ |
| sec(-θ) | Even | sec θ |
| csc(-θ) | Odd | -csc θ |

**Memory aid:** Only cosine and secant are EVEN (both have "c" and are related).

---

## 📊 Identity Summary Diagram

```
    ┌─────────────────────────────────────────────────────────────────┐
    │                    IDENTITY MAP                                 │
    ├─────────────────────────────────────────────────────────────────┤
    │                                                                 │
    │                         sin θ                                   │
    │                        ╱     ╲                                  │
    │                       ╱       ╲                                 │
    │              tan θ = ╱         ╲ = 1/csc θ                      │
    │                     ╱           ╲                               │
    │                    ╱             ╲                              │
    │              cos θ ───────────────── 1                          │
    │                    ╲             ╱                              │
    │                     ╲           ╱                               │
    │              cot θ = ╲         ╱ = 1/sec θ                      │
    │                       ╲       ╱                                 │
    │                        ╲     ╱                                  │
    │                      sin²θ + cos²θ = 1                          │
    │                                                                 │
    └─────────────────────────────────────────────────────────────────┘
```

---

## 🌍 Real-World Applications

### 1. Physics - Wave Motion
Identities simplify expressions for wave interference and superposition.

### 2. Engineering - AC Circuits
Power calculations use sin²θ + cos²θ = 1 extensively.

### 3. Computer Graphics
Rotation matrices use trigonometric identities for efficient computation.

### 4. Signal Processing
Fourier analysis relies heavily on trigonometric identities.

---

## 📋 Summary Table

### All Fundamental Identities at a Glance

| Category | Identity |
|----------|----------|
| **Reciprocal** | csc θ = 1/sin θ |
| | sec θ = 1/cos θ |
| | cot θ = 1/tan θ |
| **Quotient** | tan θ = sin θ/cos θ |
| | cot θ = cos θ/sin θ |
| **Pythagorean** | sin²θ + cos²θ = 1 |
| | 1 + tan²θ = sec²θ |
| | 1 + cot²θ = csc²θ |
| **Co-function** | sin(90°-θ) = cos θ |
| | tan(90°-θ) = cot θ |
| **Even-Odd** | sin(-θ) = -sin θ |
| | cos(-θ) = cos θ |

### Quick Reference for Simplification

| Expression | Simplifies To |
|------------|---------------|
| sin θ · csc θ | 1 |
| cos θ · sec θ | 1 |
| tan θ · cot θ | 1 |
| sin²θ + cos²θ | 1 |
| sec²θ - tan²θ | 1 |
| csc²θ - cot²θ | 1 |

---

## ❓ Quick Revision Questions

1. **Write the three Pythagorean identities.**

2. **Simplify: $\frac{\sin \theta}{\csc \theta} + \frac{\cos \theta}{\sec \theta}$**

3. **If cos θ = -5/13 and θ is in Quadrant II, find sin θ and tan θ.**

4. **Express sec²θ - 1 as a single trigonometric function squared.**

5. **Verify that csc θ - sin θ = cot θ · cos θ.**

6. **Why is sin(-30°) = -sin(30°) but cos(-30°) = cos(30°)?**

<details>
<summary>Click to see answers</summary>

1. sin²θ + cos²θ = 1  
   1 + tan²θ = sec²θ  
   1 + cot²θ = csc²θ

2. $\frac{\sin \theta}{\csc \theta} + \frac{\cos \theta}{\sec \theta}$  
   $= \sin \theta \cdot \sin \theta + \cos \theta \cdot \cos \theta$  
   $= \sin^2 \theta + \cos^2 \theta = \boxed{1}$

3. sin²θ = 1 - cos²θ = 1 - 25/169 = 144/169  
   sin θ = ±12/13  
   In Q II, sin is positive: **sin θ = 12/13**  
   tan θ = sin θ/cos θ = (12/13)/(-5/13) = **-12/5**

4. sec²θ - 1 = **tan²θ** (from 1 + tan²θ = sec²θ)

5. LHS = csc θ - sin θ = 1/sin θ - sin θ  
   = (1 - sin²θ)/sin θ = cos²θ/sin θ  
   = (cos θ/sin θ) · cos θ = cot θ · cos θ = RHS ✓

6. Sine is an **odd function**: f(-x) = -f(x)  
   Cosine is an **even function**: f(-x) = f(x)  
   This relates to their symmetry on the unit circle.

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 2.4 Ratios for Any Angle](../02-Unit-Circle/04-ratios-for-any-angle.md) | [Unit 3 Index](README.md) | [3.2 Pythagorean Identities →](02-pythagorean-identities.md) |

---

[← Back to Main Index](../README.md)
