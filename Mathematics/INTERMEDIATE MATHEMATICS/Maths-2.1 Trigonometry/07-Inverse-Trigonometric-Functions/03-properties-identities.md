# Chapter 7.3: Properties and Identities of Inverse Trigonometric Functions

## Overview

Inverse trigonometric functions satisfy numerous important properties and identities that are essential for simplification, evaluation, and problem-solving. These identities connect inverse trig functions to each other and to algebraic expressions.

---

## 📐 Fundamental Composition Properties

### Type 1: f(f⁻¹(x)) = x

$$\sin(\sin^{-1}x) = x, \quad x \in [-1, 1]$$

$$\cos(\cos^{-1}x) = x, \quad x \in [-1, 1]$$

$$\tan(\tan^{-1}x) = x, \quad x \in \mathbb{R}$$

```
    ┌─────────────────────────────────────────────────────────────┐
    │                                                             │
    │   These work for ALL x in the DOMAIN of the inverse        │
    │                                                             │
    │   Think: "Taking inverse then applying function"           │
    │          always gets you back to where you started         │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

### Type 2: f⁻¹(f(x)) = x (with restrictions!)

$$\sin^{-1}(\sin\theta) = \theta, \quad \theta \in [-\frac{\pi}{2}, \frac{\pi}{2}]$$

$$\cos^{-1}(\cos\theta) = \theta, \quad \theta \in [0, \pi]$$

$$\tan^{-1}(\tan\theta) = \theta, \quad \theta \in (-\frac{\pi}{2}, \frac{\pi}{2})$$

```
    ⚠️ CRITICAL RESTRICTION ⚠️
    
    These only work when θ is in the RANGE of the inverse function!
    
    Example: sin⁻¹(sin(2π)) ≠ 2π  (since 2π ∉ [-π/2, π/2])
             sin⁻¹(sin(2π)) = sin⁻¹(0) = 0
```

---

## 📐 The Complementary Relationships

### Sum Equals π/2 or π

$$\boxed{\sin^{-1}x + \cos^{-1}x = \frac{\pi}{2}, \quad |x| \leq 1}$$

$$\boxed{\tan^{-1}x + \cot^{-1}x = \frac{\pi}{2}, \quad x \in \mathbb{R}}$$

$$\boxed{\sec^{-1}x + \csc^{-1}x = \frac{\pi}{2}, \quad |x| \geq 1}$$

### Proof of sin⁻¹x + cos⁻¹x = π/2

```
    Let sin⁻¹x = θ, where θ ∈ [-π/2, π/2]
    
    Then sin θ = x
    
    cos(π/2 - θ) = sin θ = x
    
    Since θ ∈ [-π/2, π/2], we have (π/2 - θ) ∈ [0, π]
    
    So π/2 - θ = cos⁻¹x
    
    Therefore: θ + cos⁻¹x = π/2
               sin⁻¹x + cos⁻¹x = π/2  ∎
```

---

## 📐 Negative Argument Properties

### Odd Functions (negation goes through)

$$\boxed{\sin^{-1}(-x) = -\sin^{-1}x}$$

$$\boxed{\tan^{-1}(-x) = -\tan^{-1}x}$$

$$\boxed{\csc^{-1}(-x) = -\csc^{-1}x}$$

### Special Functions (negation shifts by π)

$$\boxed{\cos^{-1}(-x) = \pi - \cos^{-1}x}$$

$$\boxed{\sec^{-1}(-x) = \pi - \sec^{-1}x}$$

$$\boxed{\cot^{-1}(-x) = \pi - \cot^{-1}x}$$

### Visual Understanding

```
    For odd functions:
    
         y                        sin⁻¹(-x) = -sin⁻¹x
         │                        
     π/2 ┤           *            Point (a, b) reflects to
         │         *              point (-a, -b)
       0 ┼───────*────────x
         │     *                  
    -π/2 ┤   *
         │
    
    For cos⁻¹:
    
         y                        cos⁻¹(-x) = π - cos⁻¹x
         │
       π ┤ *                      If cos⁻¹(a) = b
         │   *                    Then cos⁻¹(-a) = π - b
    π/2  ┤─────*────
         │       *                Point (a, b) relates to
       0 ┤         *              point (-a, π-b)
         └───────────── x
          -1       1
```

---

## 📐 Reciprocal Argument Identities

### For |x| ≥ 1

$$\boxed{\sin^{-1}\left(\frac{1}{x}\right) = \csc^{-1}x}$$

$$\boxed{\cos^{-1}\left(\frac{1}{x}\right) = \sec^{-1}x}$$

### For All x ≠ 0

$$\tan^{-1}\left(\frac{1}{x}\right) = \begin{cases} \cot^{-1}x = \frac{\pi}{2} - \tan^{-1}x & \text{if } x > 0 \\ \cot^{-1}x - \pi = -\frac{\pi}{2} - \tan^{-1}x & \text{if } x < 0 \end{cases}$$

### Simplified Form for x > 0

$$\boxed{\tan^{-1}x + \tan^{-1}\left(\frac{1}{x}\right) = \frac{\pi}{2}, \quad x > 0}$$

$$\boxed{\tan^{-1}x + \tan^{-1}\left(\frac{1}{x}\right) = -\frac{\pi}{2}, \quad x < 0}$$

---

## 📐 Addition Formulas for tan⁻¹

### Sum Formula

$$\boxed{\tan^{-1}x + \tan^{-1}y = \tan^{-1}\left(\frac{x+y}{1-xy}\right) + \begin{cases} 0 & \text{if } xy < 1 \\ \pi & \text{if } xy > 1, x > 0 \\ -\pi & \text{if } xy > 1, x < 0 \end{cases}}$$

### Simplified (Most Common Case: xy < 1)

$$\boxed{\tan^{-1}x + \tan^{-1}y = \tan^{-1}\left(\frac{x+y}{1-xy}\right), \quad xy < 1}$$

### Difference Formula

$$\boxed{\tan^{-1}x - \tan^{-1}y = \tan^{-1}\left(\frac{x-y}{1+xy}\right), \quad xy > -1}$$

### Proof of Sum Formula (xy < 1)

```
    Let tan⁻¹x = α and tan⁻¹y = β
    
    Then tan α = x and tan β = y
    
    tan(α + β) = (tan α + tan β)/(1 - tan α tan β)
                = (x + y)/(1 - xy)
    
    If xy < 1, then (α + β) ∈ (-π/2, π/2), so:
    
    α + β = tan⁻¹((x + y)/(1 - xy))
    
    tan⁻¹x + tan⁻¹y = tan⁻¹((x + y)/(1 - xy))  ∎
```

---

## 📐 Double and Half Value Formulas

### Double Value

$$\boxed{2\tan^{-1}x = \tan^{-1}\left(\frac{2x}{1-x^2}\right), \quad |x| < 1}$$

$$\boxed{2\tan^{-1}x = \sin^{-1}\left(\frac{2x}{1+x^2}\right), \quad |x| \leq 1}$$

$$\boxed{2\tan^{-1}x = \cos^{-1}\left(\frac{1-x^2}{1+x^2}\right), \quad x \geq 0}$$

### Derivation of 2tan⁻¹x = sin⁻¹(2x/(1+x²))

```
    Let tan⁻¹x = θ, so tan θ = x
    
    Then: sin 2θ = 2 tan θ/(1 + tan²θ) = 2x/(1 + x²)
    
    So: 2θ = sin⁻¹(2x/(1 + x²))
    
    Therefore: 2tan⁻¹x = sin⁻¹(2x/(1 + x²))  ∎
```

---

## 📐 Special Values and Sums

### Important Special Sums

$$\tan^{-1}1 + \tan^{-1}2 + \tan^{-1}3 = \pi$$

$$\tan^{-1}\frac{1}{2} + \tan^{-1}\frac{1}{3} = \frac{\pi}{4}$$

$$\tan^{-1}1 + \tan^{-1}\frac{1}{2} + \tan^{-1}\frac{1}{3} = \frac{\pi}{2}$$

### Proof: tan⁻¹(1/2) + tan⁻¹(1/3) = π/4

```
    tan⁻¹(1/2) + tan⁻¹(1/3) = tan⁻¹((1/2 + 1/3)/(1 - 1/6))
    
                            = tan⁻¹((5/6)/(5/6))
                            
                            = tan⁻¹(1)
                            
                            = π/4  ∎
```

---

## 📐 Inverse Trigonometric Functions with Algebraic Expressions

### Converting Between Forms

For x ∈ [-1, 1]:

$$\sin^{-1}x = \cos^{-1}\sqrt{1-x^2}, \quad x \geq 0$$

$$\sin^{-1}x = -\cos^{-1}\sqrt{1-x^2}, \quad x < 0$$

$$\sin^{-1}x = \tan^{-1}\frac{x}{\sqrt{1-x^2}}, \quad |x| < 1$$

### Geometric Interpretation

```
    If sin⁻¹x = θ, then sin θ = x = x/1
    
    Drawing a right triangle:
    
              /|
             / |
          1 /  | x
           /   |
          /θ   |
         /_____|
         √(1-x²)
    
    From this triangle:
    
    cos θ = √(1-x²)/1 = √(1-x²)
    tan θ = x/√(1-x²)
    
    Therefore:
    sin⁻¹x = θ = cos⁻¹(√(1-x²)) = tan⁻¹(x/√(1-x²))
```

---

## 📝 Worked Examples

### Example 1: Simplify sin⁻¹(sin 5π/6)

**Solution:**
5π/6 is NOT in [-π/2, π/2], so we can't directly say the answer is 5π/6.

```
    sin(5π/6) = sin(π - π/6) = sin(π/6) = 1/2
    
    sin⁻¹(sin 5π/6) = sin⁻¹(1/2) = π/6
```

**Answer: π/6**

---

### Example 2: Prove that tan⁻¹(1/2) + tan⁻¹(1/5) + tan⁻¹(1/8) = π/4

**Solution:**
Use the addition formula twice:

```
    First, combine tan⁻¹(1/2) + tan⁻¹(1/5):
    
    = tan⁻¹((1/2 + 1/5)/(1 - 1/10))
    = tan⁻¹((7/10)/(9/10))
    = tan⁻¹(7/9)
    
    Now add tan⁻¹(1/8):
    
    tan⁻¹(7/9) + tan⁻¹(1/8) = tan⁻¹((7/9 + 1/8)/(1 - 7/72))
    
    Numerator: 7/9 + 1/8 = 56/72 + 9/72 = 65/72
    
    Denominator: 1 - 7/72 = 65/72
    
    = tan⁻¹((65/72)/(65/72))
    = tan⁻¹(1)
    = π/4  ∎
```

---

### Example 3: Express tan⁻¹(cos x/(1 + sin x)) in simplest form

**Solution:**

```
    Use the half-angle substitution approach:
    
    cos x = cos²(x/2) - sin²(x/2)
    1 + sin x = 1 + 2 sin(x/2) cos(x/2)
              = sin²(x/2) + cos²(x/2) + 2 sin(x/2) cos(x/2)
              = (cos(x/2) + sin(x/2))²
    
    cos x/(1 + sin x) = (cos(x/2) - sin(x/2))(cos(x/2) + sin(x/2))
                        ─────────────────────────────────────────────
                                (cos(x/2) + sin(x/2))²
    
                      = (cos(x/2) - sin(x/2))/(cos(x/2) + sin(x/2))
    
    Divide numerator and denominator by cos(x/2):
    
                      = (1 - tan(x/2))/(1 + tan(x/2))
    
                      = tan(π/4 - x/2)
```

**Answer: tan⁻¹(cos x/(1 + sin x)) = π/4 - x/2**

---

### Example 4: If sin⁻¹x + sin⁻¹y = π/2, prove x² + y² = 1

**Solution:**

```
    sin⁻¹x + sin⁻¹y = π/2
    
    sin⁻¹y = π/2 - sin⁻¹x = cos⁻¹x    [using the complementary identity]
    
    y = sin(cos⁻¹x)
    
    If cos⁻¹x = θ, then cos θ = x
    sin θ = √(1 - cos²θ) = √(1 - x²)
    
    So y = √(1 - x²)
    
    Therefore: y² = 1 - x²
    
    x² + y² = 1  ∎
```

---

## 📋 Summary Table

### Key Identities

| Identity | Condition | Notes |
|----------|-----------|-------|
| sin⁻¹x + cos⁻¹x = π/2 | \|x\| ≤ 1 | Complementary pair |
| tan⁻¹x + cot⁻¹x = π/2 | x ∈ ℝ | Complementary pair |
| sin⁻¹(-x) = -sin⁻¹x | \|x\| ≤ 1 | Odd function |
| cos⁻¹(-x) = π - cos⁻¹x | \|x\| ≤ 1 | Shift property |
| tan⁻¹(-x) = -tan⁻¹x | x ∈ ℝ | Odd function |
| tan⁻¹x + tan⁻¹(1/x) = π/2 | x > 0 | Reciprocal sum |
| tan⁻¹x + tan⁻¹y = tan⁻¹((x+y)/(1-xy)) | xy < 1 | Addition formula |
| 2tan⁻¹x = tan⁻¹(2x/(1-x²)) | \|x\| < 1 | Double angle |

### Composition Rules

| Composition | Result | Condition |
|-------------|--------|-----------|
| sin(sin⁻¹x) | x | \|x\| ≤ 1 |
| sin⁻¹(sin θ) | θ | θ ∈ [-π/2, π/2] |
| cos(cos⁻¹x) | x | \|x\| ≤ 1 |
| cos⁻¹(cos θ) | θ | θ ∈ [0, π] |
| tan(tan⁻¹x) | x | x ∈ ℝ |
| tan⁻¹(tan θ) | θ | θ ∈ (-π/2, π/2) |

---

## ❓ Quick Revision Questions

1. **Evaluate: cos⁻¹(cos 4π/3)**

2. **Simplify: tan⁻¹(1) + tan⁻¹(2) + tan⁻¹(3)**

3. **If tan⁻¹x + tan⁻¹y = π/4, express y in terms of x.**

4. **Prove: sin⁻¹(3/5) + sin⁻¹(4/5) = π/2**

5. **Evaluate: tan⁻¹(1/2) + tan⁻¹(1/3)**

6. **Express 2tan⁻¹(1/3) as a single inverse function.**

<details>
<summary>Click to see answers</summary>

1. **cos⁻¹(cos 4π/3)**
   
   4π/3 is NOT in [0, π], so we need to adjust.
   
   cos(4π/3) = cos(π + π/3) = -cos(π/3) = -1/2
   
   cos⁻¹(-1/2) = **2π/3**

2. **tan⁻¹(1) + tan⁻¹(2) + tan⁻¹(3)**
   
   First: tan⁻¹(1) + tan⁻¹(2) = π/4 + tan⁻¹(2)
   
   = tan⁻¹((1+2)/(1-2)) + π  [since xy = 2 > 1 and x > 0]
   = tan⁻¹(-3) + π
   = π - tan⁻¹(3)
   
   Adding tan⁻¹(3):
   π - tan⁻¹(3) + tan⁻¹(3) = **π**

3. **tan⁻¹x + tan⁻¹y = π/4**
   
   tan⁻¹y = π/4 - tan⁻¹x = tan⁻¹((1-x)/(1+x))  [assuming 1·x < 1]
   
   **y = (1-x)/(1+x)**

4. **sin⁻¹(3/5) + sin⁻¹(4/5) = π/2**
   
   Note that (3/5)² + (4/5)² = 9/25 + 16/25 = 25/25 = 1
   
   So 4/5 = √(1 - (3/5)²), which means:
   sin⁻¹(4/5) = cos⁻¹(3/5)
   
   Therefore: sin⁻¹(3/5) + sin⁻¹(4/5) = sin⁻¹(3/5) + cos⁻¹(3/5) = **π/2** ∎

5. **tan⁻¹(1/2) + tan⁻¹(1/3)**
   
   = tan⁻¹((1/2 + 1/3)/(1 - 1/6))
   = tan⁻¹((5/6)/(5/6))
   = tan⁻¹(1)
   = **π/4**

6. **2tan⁻¹(1/3)**
   
   Using 2tan⁻¹x = tan⁻¹(2x/(1-x²)):
   
   = tan⁻¹(2(1/3)/(1 - 1/9))
   = tan⁻¹((2/3)/(8/9))
   = tan⁻¹((2/3)(9/8))
   = **tan⁻¹(3/4)**

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 7.2 Graphs of Inverse Functions](02-graphs.md) | [Unit 7 Index](README.md) | [7.4 Applications →](04-applications.md) |

---

[← Back to Main Index](../README.md)
