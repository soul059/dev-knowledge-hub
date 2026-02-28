# Chapter 4: Trigonometric Substitutions

[← Previous: Partial Fractions](03-partial-fractions.md) | [Back to README](../README.md) | [Next: Weierstrass Substitution →](05-weierstrass-substitution.md)

---

## 4.1 Overview

**Trigonometric substitution** is used when the integrand contains expressions of the form:

- √(a² − x²)
- √(x² + a²)
- √(x² − a²)

The idea is to use a **trigonometric identity** to eliminate the square root, converting the integral into a purely trigonometric one.

---

## 4.2 The Three Standard Substitutions

### Master Table

| Expression | Substitution | Identity Used | Range |
|---|---|---|---|
| √(a² − x²) | x = a sin θ | 1 − sin²θ = cos²θ | −π/2 ≤ θ ≤ π/2 |
| √(x² + a²) | x = a tan θ | 1 + tan²θ = sec²θ | −π/2 < θ < π/2 |
| √(x² − a²) | x = a sec θ | sec²θ − 1 = tan²θ | 0 ≤ θ < π/2 or π ≤ θ < 3π/2 |

### How Each Substitution Eliminates the Radical

```
  ┌─────────────────────────────────────────────────────────┐
  │  Expression        Substitution    Simplification       │
  │  ──────────        ────────────    ──────────────       │
  │  √(a² − x²)       x = a sin θ    = a√(1−sin²θ)       │
  │                                    = a cos θ            │
  │                                                         │
  │  √(x² + a²)       x = a tan θ    = a√(1+tan²θ)       │
  │                                    = a sec θ            │
  │                                                         │
  │  √(x² − a²)       x = a sec θ    = a√(sec²θ−1)       │
  │                                    = a tan θ            │
  └─────────────────────────────────────────────────────────┘
```

---

## 4.3 Reference Triangles

For each substitution, draw a **right triangle** to convert back from θ to x.

### For x = a sin θ (so sin θ = x/a)

```
         ╱│
    a   ╱ │
       ╱  │ x
      ╱   │
     ╱ θ  │
    ╱─────┘
     √(a²−x²)

  sin θ = x/a       cos θ = √(a²−x²)/a
  tan θ = x/√(a²−x²)
```

### For x = a tan θ (so tan θ = x/a)

```
         ╱│
 √(x²+a²)│
       ╱  │ x
      ╱   │
     ╱ θ  │
    ╱─────┘
       a

  sin θ = x/√(x²+a²)    cos θ = a/√(x²+a²)
  tan θ = x/a
```

### For x = a sec θ (so sec θ = x/a)

```
         ╱│
    x   ╱ │
       ╱  │ √(x²−a²)
      ╱   │
     ╱ θ  │
    ╱─────┘
       a

  cos θ = a/x         sin θ = √(x²−a²)/x
  tan θ = √(x²−a²)/a
```

---

## 4.4 Step-by-Step Procedure

```
  Step 1: Identify the form √(a²−x²), √(x²+a²), or √(x²−a²)
     │
     ▼
  Step 2: Apply the corresponding substitution
     │    Also compute dx in terms of dθ
     ▼
  Step 3: Substitute into the integral and simplify
     │    The square root should disappear
     ▼
  Step 4: Evaluate the resulting trigonometric integral
     │
     ▼
  Step 5: Draw the reference triangle
     │    Convert back from θ to x
     ▼
  Step 6: Write final answer in terms of x + C
```

---

## 4.5 Worked Examples

### Example 1: ∫ dx / √(4 − x²)

Here a² − x² with a = 2.

**Substitution:** x = 2 sin θ, dx = 2 cos θ dθ

$$\int \frac{2\cos\theta\, d\theta}{\sqrt{4 - 4\sin^2\theta}} = \int \frac{2\cos\theta\, d\theta}{2\cos\theta} = \int d\theta = \theta + C$$

Back-substitute: θ = sin⁻¹(x/2)

$$= \sin^{-1}\left(\frac{x}{2}\right) + C$$

---

### Example 2: ∫ x²/√(9 − x²) dx

Here a = 3. Let x = 3 sin θ, dx = 3 cos θ dθ.

$$\int \frac{9\sin^2\theta}{3\cos\theta} \cdot 3\cos\theta\, d\theta = 9\int \sin^2\theta\, d\theta$$

$$= 9 \cdot \frac{1}{2}(\theta - \sin\theta\cos\theta) + C$$

Using the reference triangle (sin θ = x/3, cos θ = √(9−x²)/3):

$$= \frac{9}{2}\sin^{-1}\left(\frac{x}{3}\right) - \frac{x\sqrt{9-x^2}}{2} + C$$

---

### Example 3: ∫ dx / (x² + 4)^(3/2)

Here x² + a² form with a = 2. Let x = 2 tan θ, dx = 2 sec²θ dθ.

$$x^2 + 4 = 4\tan^2\theta + 4 = 4\sec^2\theta$$

$$(x^2+4)^{3/2} = 8\sec^3\theta$$

$$\int \frac{2\sec^2\theta\, d\theta}{8\sec^3\theta} = \frac{1}{4}\int \cos\theta\, d\theta = \frac{1}{4}\sin\theta + C$$

From the triangle: sin θ = x/√(x²+4)

$$= \frac{x}{4\sqrt{x^2+4}} + C$$

---

### Example 4: ∫ √(x² − 9) / x dx

Here x² − a² with a = 3. Let x = 3 sec θ, dx = 3 sec θ tan θ dθ.

$$\sqrt{x^2-9} = 3\tan\theta$$

$$\int \frac{3\tan\theta}{3\sec\theta} \cdot 3\sec\theta\tan\theta\, d\theta = 3\int \tan^2\theta\, d\theta$$

$$= 3\int(\sec^2\theta - 1)\, d\theta = 3(\tan\theta - \theta) + C$$

From the triangle: tan θ = √(x²−9)/3, θ = sec⁻¹(x/3)

$$= \sqrt{x^2-9} - 3\sec^{-1}\left(\frac{x}{3}\right) + C$$

---

### Example 5: Completing the Square + Trig Sub

$$\int \frac{dx}{\sqrt{x^2 + 6x + 13}}$$

**Complete the square:** x² + 6x + 13 = (x+3)² + 4

Let u = x + 3, then du = dx:

$$\int \frac{du}{\sqrt{u^2 + 4}}$$

Now let u = 2 tan θ, du = 2 sec²θ dθ:

$$\int \frac{2\sec^2\theta\, d\theta}{2\sec\theta} = \int \sec\theta\, d\theta = \ln|\sec\theta + \tan\theta| + C$$

From triangle: tan θ = u/2, sec θ = √(u²+4)/2

$$= \ln\left|\frac{\sqrt{u^2+4}}{2} + \frac{u}{2}\right| + C = \ln\left|u + \sqrt{u^2+4}\right| + C'$$

$$= \ln\left|x + 3 + \sqrt{x^2+6x+13}\right| + C$$

---

## 4.6 Common Trigonometric Integrals You'll Encounter

After substitution, you often need these:

| Integral | Result |
|---|---|
| ∫ sin²θ dθ | (θ − sin θ cos θ)/2 + C |
| ∫ cos²θ dθ | (θ + sin θ cos θ)/2 + C |
| ∫ tan²θ dθ | tan θ − θ + C |
| ∫ sec θ dθ | ln\|sec θ + tan θ\| + C |
| ∫ sec³θ dθ | (sec θ tan θ + ln\|sec θ + tan θ\|)/2 + C |

---

## 4.7 Decision Flowchart — Which Technique?

```
  Integral contains a square root?
     │
     ├── √(a² − x²) ──► x = a sin θ
     │
     ├── √(x² + a²) ──► x = a tan θ
     │
     ├── √(x² − a²) ──► x = a sec θ
     │
     ├── √(ax² + bx + c) ──► Complete the square first
     │                         then use one of the above
     │
     └── No square root with x² ± a² form
         ──► Try other methods (substitution, parts, etc.)
```

---

## Summary Table

| Concept | Key Point |
|---|---|
| When to use | Integrands with √(a²−x²), √(x²+a²), √(x²−a²) |
| √(a²−x²) | x = a sin θ; uses cos²θ = 1 − sin²θ |
| √(x²+a²) | x = a tan θ; uses sec²θ = 1 + tan²θ |
| √(x²−a²) | x = a sec θ; uses tan²θ = sec²θ − 1 |
| Complete the square | Use when quadratic under root is not in standard form |
| Reference triangle | Essential for back-substitution from θ to x |
| After substitution | Must evaluate a trig integral (often sin², cos², etc.) |

---

## Quick Revision Questions

1. Which substitution would you use for ∫ √(25 − x²) dx? Set up but don't need to fully evaluate.

2. Draw the reference triangle for x = 5 tan θ and express sin θ in terms of x.

3. Evaluate ∫ dx / √(x² + 1) using trigonometric substitution.

4. Why must you complete the square before applying a trig substitution to ∫ dx / √(x² + 4x + 8)?

5. Evaluate ∫ x² √(1 − x²) dx using x = sin θ.

6. After the substitution x = a sec θ, what is dx in terms of θ?

---

[← Previous: Partial Fractions](03-partial-fractions.md) | [Back to README](../README.md) | [Next: Weierstrass Substitution →](05-weierstrass-substitution.md)
