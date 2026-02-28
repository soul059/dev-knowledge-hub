# Chapter 7.1: Definition and Principal Values

## Overview

Inverse trigonometric functions answer the question: "What angle gives this ratio?" Since trigonometric functions are periodic and not one-to-one, we must restrict their domains to create proper inverse functions. This chapter explains these restrictions and the resulting **principal value** ranges.

---

## 📐 The Need for Inverse Functions

### The Fundamental Question

```
    If sin θ = 0.5, what is θ?
    
    Possible answers: 30°, 150°, 390°, 510°, -210°, ...
                      (infinitely many!)
    
    We need ONE unique answer → Principal Value
    
    Principal value of sin⁻¹(0.5) = 30° = π/6
```

### Creating Inverse Functions

For a function to have an inverse, it must be **one-to-one** (each output comes from exactly one input).

Trigonometric functions are NOT one-to-one over their full domains, so we restrict their domains.

---

## 📐 The Three Main Inverse Functions

### 1. Inverse Sine: sin⁻¹(x) or arcsin(x)

$$\boxed{y = \sin^{-1}(x) \iff x = \sin(y), \quad -\frac{\pi}{2} \leq y \leq \frac{\pi}{2}}$$

| Property | Value |
|----------|-------|
| Domain | [-1, 1] |
| Range (Principal Values) | [-π/2, π/2] or [-90°, 90°] |
| Quadrants covered | Q4 (negative angles) and Q1 |

```
    Why this range for arcsin?
    
    In [-π/2, π/2], sine goes from -1 to 1,
    covering ALL possible sine values exactly once.
    
         sin x
           │
         1 ┤         *
           │       *
           │     *
         0 ┼───*─────────
           │ *
           │*
        -1 ┤*
           └──────────────
          -π/2   0   π/2
           
    This is the restricted domain where sin is one-to-one.
```

### 2. Inverse Cosine: cos⁻¹(x) or arccos(x)

$$\boxed{y = \cos^{-1}(x) \iff x = \cos(y), \quad 0 \leq y \leq \pi}$$

| Property | Value |
|----------|-------|
| Domain | [-1, 1] |
| Range (Principal Values) | [0, π] or [0°, 180°] |
| Quadrants covered | Q1 and Q2 |

```
    Why this range for arccos?
    
    In [0, π], cosine goes from 1 to -1,
    covering ALL possible cosine values exactly once.
    
         cos x
           │
         1 *
           │ *
           │   *
         0 ┼─────*────────
           │       *
           │         *
        -1 ┤           *
           └──────────────
           0    π/2    π
```

### 3. Inverse Tangent: tan⁻¹(x) or arctan(x)

$$\boxed{y = \tan^{-1}(x) \iff x = \tan(y), \quad -\frac{\pi}{2} < y < \frac{\pi}{2}}$$

| Property | Value |
|----------|-------|
| Domain | (-∞, ∞) |
| Range (Principal Values) | (-π/2, π/2) or (-90°, 90°) |
| Quadrants covered | Q4 and Q1 |

```
    Why this range for arctan?
    
    In (-π/2, π/2), tangent goes from -∞ to +∞,
    covering ALL real numbers exactly once.
    
         tan x
           │        :
       +∞  ┤       ╱:
           │      ╱ :
         0 ┼─────╱──:
           │    ╱   :
       -∞  ┤   ╱    :
           └────────:────
          -π/2   0  π/2
           
    (Note: asymptotes at ±π/2, open interval)
```

---

## 📊 Summary of Principal Value Ranges

```
    ┌────────────────────────────────────────────────────────────────┐
    │                  PRINCIPAL VALUE RANGES                        │
    ├────────────────────────────────────────────────────────────────┤
    │                                                                │
    │  Function      Domain          Range              Notation     │
    │  ────────      ──────          ─────              ────────     │
    │                                                                │
    │  sin⁻¹(x)     [-1, 1]         [-π/2, π/2]        arcsin      │
    │                                                                │
    │  cos⁻¹(x)     [-1, 1]         [0, π]             arccos      │
    │                                                                │
    │  tan⁻¹(x)     (-∞, ∞)         (-π/2, π/2)        arctan      │
    │                                                                │
    │  cot⁻¹(x)     (-∞, ∞)         (0, π)             arccot      │
    │                                                                │
    │  sec⁻¹(x)     |x| ≥ 1         [0, π], y ≠ π/2   arcsec      │
    │                                                                │
    │  csc⁻¹(x)     |x| ≥ 1         [-π/2, π/2], y≠0  arccsc      │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 🧮 Evaluating Inverse Functions

### Standard Values

| x | sin⁻¹(x) | cos⁻¹(x) |
|---|----------|----------|
| 0 | 0 | π/2 |
| 1/2 | π/6 | π/3 |
| √2/2 | π/4 | π/4 |
| √3/2 | π/3 | π/6 |
| 1 | π/2 | 0 |
| -1/2 | -π/6 | 2π/3 |
| -√2/2 | -π/4 | 3π/4 |
| -√3/2 | -π/3 | 5π/6 |
| -1 | -π/2 | π |

| x | tan⁻¹(x) |
|---|----------|
| 0 | 0 |
| 1/√3 | π/6 |
| 1 | π/4 |
| √3 | π/3 |
| -1/√3 | -π/6 |
| -1 | -π/4 |
| -√3 | -π/3 |

---

## 🧮 Worked Examples

### Example 1: Evaluate sin⁻¹(√3/2)

**Solution:**
We need θ such that sin θ = √3/2 and θ ∈ [-π/2, π/2].

sin(π/3) = √3/2 and π/3 ∈ [-π/2, π/2] ✓

$$\sin^{-1}\left(\frac{\sqrt{3}}{2}\right) = \boxed{\frac{\pi}{3}}$$

### Example 2: Evaluate cos⁻¹(-1/2)

**Solution:**
We need θ such that cos θ = -1/2 and θ ∈ [0, π].

cos(2π/3) = -1/2 and 2π/3 ∈ [0, π] ✓

$$\cos^{-1}\left(-\frac{1}{2}\right) = \boxed{\frac{2\pi}{3}}$$

### Example 3: Evaluate tan⁻¹(-1)

**Solution:**
We need θ such that tan θ = -1 and θ ∈ (-π/2, π/2).

tan(-π/4) = -1 and -π/4 ∈ (-π/2, π/2) ✓

$$\tan^{-1}(-1) = \boxed{-\frac{\pi}{4}}$$

### Example 4: Evaluate sin⁻¹(sin(2π/3))

**Solution:**
Note: sin(2π/3) = sin(π - π/3) = sin(π/3) = √3/2

But 2π/3 is NOT in the range of sin⁻¹ (which is [-π/2, π/2]).

So we can't just say "2π/3". We need to find the angle in [-π/2, π/2] with the same sine.

sin⁻¹(√3/2) = π/3

$$\sin^{-1}(\sin(2\pi/3)) = \boxed{\frac{\pi}{3}}$$

### Example 5: Evaluate cos⁻¹(cos(-π/4))

**Solution:**
cos(-π/4) = cos(π/4) = √2/2

Now find θ ∈ [0, π] such that cos θ = √2/2.

θ = π/4 ∈ [0, π] ✓

$$\cos^{-1}(\cos(-\pi/4)) = \boxed{\frac{\pi}{4}}$$

### Example 6: Evaluate tan⁻¹(tan(3π/4))

**Solution:**
tan(3π/4) = -1

But 3π/4 is NOT in the range of tan⁻¹ (which is (-π/2, π/2)).

Find θ ∈ (-π/2, π/2) such that tan θ = -1.

θ = -π/4 ∈ (-π/2, π/2) ✓

$$\tan^{-1}(\tan(3\pi/4)) = \boxed{-\frac{\pi}{4}}$$

---

## 📐 Reciprocal Inverse Functions

### Inverse Cotangent: cot⁻¹(x)

$$y = \cot^{-1}(x) \iff x = \cot(y), \quad 0 < y < \pi$$

### Inverse Secant: sec⁻¹(x)

$$y = \sec^{-1}(x) \iff x = \sec(y), \quad y \in [0, \pi], y \neq \frac{\pi}{2}$$

### Inverse Cosecant: csc⁻¹(x)

$$y = \csc^{-1}(x) \iff x = \csc(y), \quad y \in \left[-\frac{\pi}{2}, \frac{\pi}{2}\right], y \neq 0$$

### Relationships

$$\sec^{-1}(x) = \cos^{-1}\left(\frac{1}{x}\right)$$

$$\csc^{-1}(x) = \sin^{-1}\left(\frac{1}{x}\right)$$

$$\cot^{-1}(x) = \frac{\pi}{2} - \tan^{-1}(x) \quad \text{(for all x)}$$

---

## 📊 Important Notes

### Common Mistakes to Avoid

```
    ┌────────────────────────────────────────────────────────────────┐
    │                    COMMON MISTAKES                             │
    ├────────────────────────────────────────────────────────────────┤
    │                                                                │
    │  ✗ sin⁻¹(x) = 1/sin(x)                                        │
    │    (sin⁻¹ is inverse function, not reciprocal)               │
    │                                                                │
    │  ✗ sin⁻¹(sin θ) = θ always                                    │
    │    (Only true when θ is in principal range)                   │
    │                                                                │
    │  ✗ sin⁻¹(2) = some angle                                      │
    │    (2 is outside the domain [-1, 1])                          │
    │                                                                │
    │  ✗ Confusing ranges: sin⁻¹ range is NOT [0, π]               │
    │    (That's for cos⁻¹; sin⁻¹ range is [-π/2, π/2])            │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

### When f⁻¹(f(x)) = x

| Function | f⁻¹(f(x)) = x when... |
|----------|----------------------|
| sin⁻¹(sin x) = x | x ∈ [-π/2, π/2] |
| cos⁻¹(cos x) = x | x ∈ [0, π] |
| tan⁻¹(tan x) = x | x ∈ (-π/2, π/2) |

### When f(f⁻¹(x)) = x

| Function | f(f⁻¹(x)) = x when... |
|----------|----------------------|
| sin(sin⁻¹ x) = x | x ∈ [-1, 1] (always true in domain) |
| cos(cos⁻¹ x) = x | x ∈ [-1, 1] (always true in domain) |
| tan(tan⁻¹ x) = x | x ∈ ℝ (always true) |

---

## 📋 Summary Table

### Quick Reference

| Function | Domain | Range |
|----------|--------|-------|
| sin⁻¹(x) | [-1, 1] | [-π/2, π/2] |
| cos⁻¹(x) | [-1, 1] | [0, π] |
| tan⁻¹(x) | ℝ | (-π/2, π/2) |
| cot⁻¹(x) | ℝ | (0, π) |
| sec⁻¹(x) | \|x\| ≥ 1 | [0, π], y ≠ π/2 |
| csc⁻¹(x) | \|x\| ≥ 1 | [-π/2, π/2], y ≠ 0 |

### Key Points

| Concept | Detail |
|---------|--------|
| Principal value | Unique angle in the specified range |
| sin⁻¹ range | Q4 and Q1 (symmetric about 0) |
| cos⁻¹ range | Q1 and Q2 (above x-axis) |
| tan⁻¹ range | Q4 and Q1 (excludes endpoints) |
| Notation | sin⁻¹(x) = arcsin(x) ≠ 1/sin(x) |

---

## ❓ Quick Revision Questions

1. **Evaluate: sin⁻¹(1/2)**

2. **Evaluate: cos⁻¹(-√3/2)**

3. **Evaluate: tan⁻¹(√3)**

4. **Find: sin⁻¹(sin(5π/6))**

5. **Find: cos⁻¹(cos(-π/3))**

6. **Why is sin⁻¹(2) undefined?**

<details>
<summary>Click to see answers</summary>

1. sin⁻¹(1/2) = **π/6**  
   (sin(π/6) = 1/2 and π/6 ∈ [-π/2, π/2])

2. cos⁻¹(-√3/2) = **5π/6**  
   (cos(5π/6) = -√3/2 and 5π/6 ∈ [0, π])

3. tan⁻¹(√3) = **π/3**  
   (tan(π/3) = √3 and π/3 ∈ (-π/2, π/2))

4. sin(5π/6) = sin(π - π/6) = sin(π/6) = 1/2  
   sin⁻¹(1/2) = **π/6**  
   (Note: 5π/6 is NOT the answer because it's outside [-π/2, π/2])

5. cos(-π/3) = cos(π/3) = 1/2  
   cos⁻¹(1/2) = **π/3**  
   (Note: -π/3 is NOT the answer because it's outside [0, π])

6. sin⁻¹(2) is undefined because 2 is outside the domain of sin⁻¹.  
   The domain of sin⁻¹ is [-1, 1] because the range of sin is [-1, 1].  
   No angle exists whose sine is 2 (since -1 ≤ sin θ ≤ 1 for all θ).

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Unit 6: Advanced Techniques](../06-Trigonometric-Equations/04-advanced-techniques.md) | [Unit 7 Index](README.md) | [7.2 Graphs →](02-graphs.md) |

---

[← Back to Main Index](../README.md)
