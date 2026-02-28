# Chapter 7.4: Applications of Inverse Trigonometric Functions

## Overview

Inverse trigonometric functions have wide-ranging applications in solving equations, calculus, coordinate geometry, physics, and engineering. This chapter explores practical applications and problem-solving techniques.

---

## 📐 Solving Trigonometric Equations

### Using Inverse Functions to Find Solutions

When solving equations of the form trig(x) = k, inverse functions give us the **principal value**, from which we can find all solutions.

### Example: Solve 2sin²x + 3sin x - 2 = 0

**Solution:**

```
    Let y = sin x
    
    2y² + 3y - 2 = 0
    (2y - 1)(y + 2) = 0
    
    y = 1/2  or  y = -2
    
    Since sin x ∈ [-1, 1], y = -2 is rejected.
    
    sin x = 1/2
    
    Using inverse function:
    x = sin⁻¹(1/2) = π/6  (principal value)
    
    General solution: x = nπ + (-1)ⁿ(π/6)
```

### Example: Solve tan x + cot x = 2

**Solution:**

```
    tan x + 1/tan x = 2
    
    tan²x + 1 = 2 tan x
    
    tan²x - 2 tan x + 1 = 0
    
    (tan x - 1)² = 0
    
    tan x = 1
    
    x = tan⁻¹(1) = π/4  (principal value)
    
    General solution: x = nπ + π/4
```

---

## 📐 Calculus Applications

### Derivatives of Inverse Trigonometric Functions

$$\boxed{\frac{d}{dx}(\sin^{-1}x) = \frac{1}{\sqrt{1-x^2}}}$$

$$\boxed{\frac{d}{dx}(\cos^{-1}x) = -\frac{1}{\sqrt{1-x^2}}}$$

$$\boxed{\frac{d}{dx}(\tan^{-1}x) = \frac{1}{1+x^2}}$$

$$\boxed{\frac{d}{dx}(\cot^{-1}x) = -\frac{1}{1+x^2}}$$

$$\boxed{\frac{d}{dx}(\sec^{-1}x) = \frac{1}{|x|\sqrt{x^2-1}}}$$

$$\boxed{\frac{d}{dx}(\csc^{-1}x) = -\frac{1}{|x|\sqrt{x^2-1}}}$$

### Proof: d/dx(sin⁻¹x) = 1/√(1-x²)

```
    Let y = sin⁻¹x
    
    Then sin y = x
    
    Differentiate implicitly:
    cos y · dy/dx = 1
    
    dy/dx = 1/cos y
    
    Since y ∈ [-π/2, π/2], cos y ≥ 0
    cos y = √(1 - sin²y) = √(1 - x²)
    
    Therefore: dy/dx = 1/√(1-x²)  ∎
```

---

## 📐 Integration Formulas

### Standard Integrals

$$\boxed{\int \frac{dx}{\sqrt{1-x^2}} = \sin^{-1}x + C}$$

$$\boxed{\int \frac{dx}{\sqrt{a^2-x^2}} = \sin^{-1}\frac{x}{a} + C}$$

$$\boxed{\int \frac{dx}{1+x^2} = \tan^{-1}x + C}$$

$$\boxed{\int \frac{dx}{a^2+x^2} = \frac{1}{a}\tan^{-1}\frac{x}{a} + C}$$

$$\boxed{\int \frac{dx}{x\sqrt{x^2-1}} = \sec^{-1}|x| + C}$$

### Integration Example

**Evaluate: ∫ dx/(4 + x²)**

```
    ∫ dx/(4 + x²) = ∫ dx/(2² + x²)
    
    Using the formula with a = 2:
    
    = (1/2) tan⁻¹(x/2) + C
```

---

## 📐 Simplification Techniques

### Converting to Algebraic Form

**Problem:** If y = sin⁻¹(x) + cos⁻¹(x), find dy/dx

**Solution:**

```
    We know that sin⁻¹x + cos⁻¹x = π/2  (constant)
    
    Therefore: y = π/2
    
    dy/dx = 0
```

### Triangle Method for Simplification

**Simplify: tan(sin⁻¹x)**

```
    Let θ = sin⁻¹x, so sin θ = x = x/1
    
    Draw a right triangle:
    
              /|
             / |
          1 /  | x
           /   |
          /θ   |
         /_____|
         √(1-x²)
    
    From the triangle:
    tan θ = opposite/adjacent = x/√(1-x²)
```

**Answer: tan(sin⁻¹x) = x/√(1-x²)**

---

## 📐 Proving Identities

### Example 1: Prove sin⁻¹(2x√(1-x²)) = 2sin⁻¹x for |x| ≤ 1/√2

**Proof:**

```
    Let sin⁻¹x = θ, so x = sin θ
    
    Then θ ∈ [-π/4, π/4] (since |x| ≤ 1/√2)
    
    2x√(1-x²) = 2 sin θ · √(1 - sin²θ)
               = 2 sin θ · cos θ
               = sin 2θ
    
    sin⁻¹(2x√(1-x²)) = sin⁻¹(sin 2θ)
    
    Since |x| ≤ 1/√2, we have |θ| ≤ π/4, so |2θ| ≤ π/2
    
    Therefore: sin⁻¹(sin 2θ) = 2θ = 2sin⁻¹x  ∎
```

### Example 2: Prove tan⁻¹((√(1+x²) - 1)/x) = (1/2)tan⁻¹x

**Proof:**

```
    Let tan⁻¹x = θ, so x = tan θ
    
    Then: 1 + x² = 1 + tan²θ = sec²θ
    √(1 + x²) = sec θ  (taking positive root)
    
    (√(1+x²) - 1)/x = (sec θ - 1)/tan θ
                     = (1 - cos θ)/(cos θ · tan θ)
                     = (1 - cos θ)/sin θ
    
    Using half-angle formulas:
    1 - cos θ = 2sin²(θ/2)
    sin θ = 2sin(θ/2)cos(θ/2)
    
    = 2sin²(θ/2)/(2sin(θ/2)cos(θ/2))
    = sin(θ/2)/cos(θ/2)
    = tan(θ/2)
    
    So: tan⁻¹((√(1+x²) - 1)/x) = tan⁻¹(tan(θ/2)) = θ/2 = (1/2)tan⁻¹x  ∎
```

---

## 📐 Physics Applications

### Angle of Projection

```
    In projectile motion, the angle θ for maximum range on an
    inclined plane of angle α is:
    
         θ = π/4 + α/2
    
    This involves inverse trig when finding θ from known components.
```

### Angle of Refraction (Snell's Law)

```
    When light passes from medium 1 to medium 2:
    
         n₁ sin θ₁ = n₂ sin θ₂
    
    The angle of refraction:
    
         θ₂ = sin⁻¹((n₁/n₂) sin θ₁)
    
    Example: Light enters water (n = 1.33) from air (n = 1)
             at 45°
    
         θ₂ = sin⁻¹((1/1.33) × sin 45°)
            = sin⁻¹((1/1.33) × 0.707)
            = sin⁻¹(0.532)
            ≈ 32.1°
```

### Pendulum Motion

```
    For a simple pendulum, the angular displacement satisfies:
    
         θ(t) = θ₀ cos(ωt)
    
    Time to reach angle θ:
    
         t = (1/ω) cos⁻¹(θ/θ₀)
```

---

## 📐 Engineering Applications

### AC Circuit Analysis

```
    In an RLC circuit, the phase angle φ is given by:
    
         φ = tan⁻¹((X_L - X_C)/R)
    
    Where:
    X_L = inductive reactance
    X_C = capacitive reactance
    R = resistance
    
    ┌──────────────────────────────────────────┐
    │                                          │
    │   Impedance Triangle:                    │
    │                                          │
    │              /|                          │
    │           Z / |                          │
    │            /  | X_L - X_C                │
    │           /φ  |                          │
    │          /____|                          │
    │            R                             │
    │                                          │
    │   φ = tan⁻¹((X_L - X_C)/R)              │
    │                                          │
    └──────────────────────────────────────────┘
```

### Surveying and Navigation

```
    Finding the angle of elevation to a target:
    
                     *  Target
                    /|
                   / |
                  /  | height h
                 /   |
                /θ   |
        ──────*──────+──────
          Observer   distance d
    
         θ = tan⁻¹(h/d)
```

---

## 📐 Coordinate Geometry Applications

### Finding Angles Between Lines

The angle θ between two lines with slopes m₁ and m₂:

$$\theta = \tan^{-1}\left|\frac{m_1 - m_2}{1 + m_1 m_2}\right|$$

### Example: Find the angle between y = 2x + 1 and y = 3x - 4

```
    m₁ = 2, m₂ = 3
    
    tan θ = |(2 - 3)/(1 + 6)| = |-1/7| = 1/7
    
    θ = tan⁻¹(1/7) ≈ 8.13°
```

### Angle Between Two Vectors

For vectors **a** and **b**:

$$\theta = \cos^{-1}\left(\frac{\mathbf{a} \cdot \mathbf{b}}{|\mathbf{a}||\mathbf{b}|}\right)$$

---

## 📐 Solving Systems with Inverse Trig

### Example: Solve sin⁻¹x + sin⁻¹(1-x) = cos⁻¹x

**Solution:**

```
    Using sin⁻¹x + cos⁻¹x = π/2:
    cos⁻¹x = π/2 - sin⁻¹x
    
    sin⁻¹x + sin⁻¹(1-x) = π/2 - sin⁻¹x
    
    2sin⁻¹x + sin⁻¹(1-x) = π/2
    
    sin⁻¹(1-x) = π/2 - 2sin⁻¹x
    
    Taking sine of both sides:
    1-x = sin(π/2 - 2sin⁻¹x) = cos(2sin⁻¹x)
    
    Let sin⁻¹x = θ, so x = sin θ
    
    1-x = cos 2θ = 1 - 2sin²θ = 1 - 2x²
    
    1 - x = 1 - 2x²
    2x² - x = 0
    x(2x - 1) = 0
    
    x = 0  or  x = 1/2
    
    Checking x = 0: sin⁻¹(0) + sin⁻¹(1) = 0 + π/2 = π/2 = cos⁻¹(0) ✓
    Checking x = 1/2: sin⁻¹(1/2) + sin⁻¹(1/2) = π/6 + π/6 = π/3
                      cos⁻¹(1/2) = π/3 ✓
```

**Answer: x = 0 or x = 1/2**

---

## 📝 Worked Examples

### Example 1: Find the value of tan(2tan⁻¹(1/5) - π/4)

**Solution:**

```
    Let α = 2tan⁻¹(1/5)
    
    Using 2tan⁻¹x = tan⁻¹(2x/(1-x²)):
    α = tan⁻¹(2(1/5)/(1 - 1/25))
      = tan⁻¹((2/5)/(24/25))
      = tan⁻¹((2/5)(25/24))
      = tan⁻¹(5/12)
    
    Now: tan(α - π/4) = (tan α - 1)/(1 + tan α)
                      = (5/12 - 1)/(1 + 5/12)
                      = (-7/12)/(17/12)
                      = -7/17
```

**Answer: -7/17**

---

### Example 2: If tan⁻¹a + tan⁻¹b + tan⁻¹c = π, prove that a + b + c = abc

**Solution:**

```
    tan⁻¹a + tan⁻¹b + tan⁻¹c = π
    
    tan⁻¹a + tan⁻¹b = π - tan⁻¹c
    
    Taking tangent of both sides:
    
    tan(tan⁻¹a + tan⁻¹b) = tan(π - tan⁻¹c)
    
    (a + b)/(1 - ab) = -c    [since tan(π - θ) = -tan θ]
    
    a + b = -c(1 - ab)
    a + b = -c + abc
    
    a + b + c = abc  ∎
```

---

## 📋 Summary Table

### Key Applications

| Area | Application | Formula |
|------|-------------|---------|
| Calculus | Derivative | d/dx(sin⁻¹x) = 1/√(1-x²) |
| Calculus | Integral | ∫dx/(1+x²) = tan⁻¹x + C |
| Geometry | Angle between lines | θ = tan⁻¹\|(m₁-m₂)/(1+m₁m₂)\| |
| Physics | Refraction | θ₂ = sin⁻¹((n₁/n₂)sin θ₁) |
| Engineering | Phase angle | φ = tan⁻¹((X_L-X_C)/R) |
| Navigation | Elevation angle | θ = tan⁻¹(h/d) |

### Derivative Memory Aid

```
    ┌────────────────────────────────────────────────────────┐
    │                                                        │
    │  "Arc-sine and arc-cosine have √(1-x²) denominators"  │
    │  "Arc-tan and arc-cot have (1+x²) denominators"       │
    │  "Arc-sec and arc-csc have |x|√(x²-1) denominators"   │
    │                                                        │
    │  sin⁻¹ and tan⁻¹ are POSITIVE                         │
    │  cos⁻¹, cot⁻¹, and csc⁻¹ are NEGATIVE                │
    │  sec⁻¹ is POSITIVE                                    │
    │                                                        │
    └────────────────────────────────────────────────────────┘
```

---

## ❓ Quick Revision Questions

1. **Find d/dx(tan⁻¹(sin x))**

2. **Evaluate: ∫₀¹ dx/(1+x²)**

3. **Find the angle between the lines y = x and y = √3x**

4. **If tan⁻¹x + tan⁻¹y + tan⁻¹z = π/2, prove that xy + yz + zx = 1**

5. **Simplify: cos(tan⁻¹(3/4))**

6. **A ladder leans against a wall with its foot 4m from the wall. If the ladder is 5m long, find the angle of inclination using inverse trig.**

<details>
<summary>Click to see answers</summary>

1. **d/dx(tan⁻¹(sin x))**
   
   Using chain rule:
   = 1/(1 + sin²x) · cos x
   = **cos x/(1 + sin²x)**

2. **∫₀¹ dx/(1+x²)**
   
   = [tan⁻¹x]₀¹
   = tan⁻¹(1) - tan⁻¹(0)
   = π/4 - 0
   = **π/4**

3. **Angle between y = x and y = √3x**
   
   m₁ = 1, m₂ = √3
   
   tan θ = |(1 - √3)/(1 + √3)|
   
   Rationalizing: = |(1 - √3)²/(1 - 3)| = |(1 - 2√3 + 3)/(-2)|
                 = |4 - 2√3|/2 = 2 - √3
   
   θ = tan⁻¹(2 - √3) = **π/12** (or 15°)

4. **tan⁻¹x + tan⁻¹y + tan⁻¹z = π/2**
   
   tan⁻¹x + tan⁻¹y = π/2 - tan⁻¹z = cot⁻¹z = tan⁻¹(1/z)
   
   Taking tangent: (x + y)/(1 - xy) = 1/z
   
   z(x + y) = 1 - xy
   xz + yz = 1 - xy
   **xy + yz + zx = 1** ∎

5. **cos(tan⁻¹(3/4))**
   
   Draw a triangle with opposite = 3, adjacent = 4
   Hypotenuse = √(9 + 16) = 5
   
   cos θ = adjacent/hypotenuse = **4/5**

6. **Ladder problem**
   
   Height on wall = √(5² - 4²) = √9 = 3m
   
   Angle of inclination θ:
   tan θ = 3/4, so θ = **tan⁻¹(3/4) ≈ 36.87°**
   
   Or: sin θ = 3/5, so θ = **sin⁻¹(3/5) ≈ 36.87°**

</details>

---

## Navigation

| Previous | Up | Next Unit |
|----------|-------|-----------|
| [← 7.3 Properties and Identities](03-properties-identities.md) | [Unit 7 Index](README.md) | [Unit 8: Properties of Triangles →](../08-Properties-of-Triangles/README.md) |

---

[← Back to Main Index](../README.md)
