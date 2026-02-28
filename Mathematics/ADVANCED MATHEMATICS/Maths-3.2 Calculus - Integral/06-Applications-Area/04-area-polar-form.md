# Chapter 4: Area in Polar Form

[← Previous: Area in Parametric Form](03-area-parametric-form.md) | [Back to README](../README.md) | [Next: Disk Method →](../07-Applications-Volume/01-disk-method.md)

---

## 4.1 Overview

Polar coordinates (r, θ) are natural for curves with rotational symmetry — circles, spirals, roses, cardioids. The area formula in polar form uses **angular sectors** instead of rectangular strips.

---

## 4.2 The Polar Area Formula

> **Theorem:** The area enclosed by r = f(θ) from θ = α to θ = β is:
>
> $$A = \frac{1}{2}\int_{\alpha}^{\beta} r^2\, d\theta = \frac{1}{2}\int_{\alpha}^{\beta} [f(\theta)]^2\, d\theta$$

### Derivation

```
  A sector with angle dθ and radius r:
  
        ╱│
       ╱ │
    r ╱  │   Area of sector ≈ ½ r² dθ
     ╱ dθ│   (like a thin triangle)
    ╱────│
    
  A = ∫ dA = ∫_α^β ½ r² dθ
```

A thin sector with radius r and angle dθ has area:

$$dA = \frac{1}{2}r^2\, d\theta$$

(This comes from the sector area formula: A = ½r²θ for a sector of angle θ.)

Integrating:

$$A = \int_{\alpha}^{\beta} \frac{1}{2}r^2\, d\theta$$

---

## 4.3 Area Between Two Polar Curves

If r₁(θ) ≤ r₂(θ) on [α, β], the area between them is:

$$A = \frac{1}{2}\int_{\alpha}^{\beta} [r_2^2 - r_1^2]\, d\theta$$

```
      ╱‾‾‾╲  r₂(θ) — outer
     ╱ ╱‾╲ ╲
    ╱ ╱███╲ ╲   ← shaded = area between
    ╲ ╲███╱ ╱
     ╲ ╲_╱ ╱  r₁(θ) — inner
      ╲___╱
```

---

## 4.4 Worked Examples

### Example 1: Circle

r = a (constant radius).

$$A = \frac{1}{2}\int_0^{2\pi} a^2\, d\theta = \frac{a^2}{2} \cdot 2\pi = \pi a^2 \quad✓$$

---

### Example 2: Cardioid

r = a(1 + cos θ), 0 ≤ θ ≤ 2π.

```
       ╱‾‾‾╲
      ╱     ╲
     ╱       ╲
    ●         ╲    Cardioid
     ╲       ╱
      ╲     ╱
       ╲___╱
```

$$A = \frac{1}{2}\int_0^{2\pi} a^2(1 + \cos\theta)^2\, d\theta$$

Expand: $(1 + \cos\theta)^2 = 1 + 2\cos\theta + \cos^2\theta$

$$= \frac{a^2}{2}\int_0^{2\pi} \left(1 + 2\cos\theta + \frac{1 + \cos 2\theta}{2}\right)\, d\theta$$

$$= \frac{a^2}{2}\int_0^{2\pi} \left(\frac{3}{2} + 2\cos\theta + \frac{\cos 2\theta}{2}\right)\, d\theta$$

$$= \frac{a^2}{2}\left[\frac{3\theta}{2} + 2\sin\theta + \frac{\sin 2\theta}{4}\right]_0^{2\pi} = \frac{a^2}{2} \cdot 3\pi$$

$$\boxed{A_{\text{cardioid}} = \frac{3\pi a^2}{2}}$$

---

### Example 3: Rose Curve

r = a sin(2θ) — a **4-petaled rose**.

```
      │  ╱╲
      │ ╱  ╲
   ╲  │╱    
    ╲ │     ╱    4 petals
  ───●●●●●●───
    ╱ │     ╲
   ╱  │╲
      │ ╲  ╱
      │  ╲╱
```

Each petal occupies θ ∈ [0, π/2] (where sin 2θ ≥ 0). Total area = 4 × one petal:

$$A = 4 \cdot \frac{1}{2}\int_0^{\pi/2} a^2 \sin^2(2\theta)\, d\theta = 2a^2 \int_0^{\pi/2} \frac{1 - \cos 4\theta}{2}\, d\theta$$

$$= a^2\left[\theta - \frac{\sin 4\theta}{4}\right]_0^{\pi/2} = a^2 \cdot \frac{\pi}{2}$$

$$\boxed{A_{\text{4-petal rose}} = \frac{\pi a^2}{2}}$$

For r = a sin(nθ):
- n even: 2n petals, each petal area = πa²/(4n)
- n odd: n petals, each petal area = πa²/(4n)

---

### Example 4: Lemniscate

r² = a² cos 2θ — exists only where cos 2θ ≥ 0.

```
       ╱╲    ╱╲
      ╱  ╲  ╱  ╲
     ╱    ╲╱    ╲     Lemniscate (figure-eight)
     ╲    ╱╲    ╱
      ╲  ╱  ╲  ╱
       ╲╱    ╲╱
```

By symmetry, total area = 4 × (one quarter, θ from 0 to π/4):

$$A = 4 \cdot \frac{1}{2}\int_0^{\pi/4} a^2 \cos 2\theta\, d\theta = 2a^2\left[\frac{\sin 2\theta}{2}\right]_0^{\pi/4} = 2a^2 \cdot \frac{1}{2} = a^2$$

$$\boxed{A_{\text{lemniscate}} = a^2}$$

---

### Example 5: Area Between Two Curves

Find the area inside r = 2 and outside r = 2(1 − cos θ).

The circle r = 2 and cardioid r = 2(1 − cos θ) intersect where:
2 = 2(1 − cos θ) → cos θ = 0 → θ = ±π/2

By symmetry (about x-axis):

$$A = 2 \cdot \frac{1}{2}\int_0^{\pi/2} [4 - 4(1 - \cos\theta)^2]\, d\theta$$

$$= \int_0^{\pi/2} 4[1 - (1 - 2\cos\theta + \cos^2\theta)]\, d\theta$$

$$= \int_0^{\pi/2} 4(2\cos\theta - \cos^2\theta)\, d\theta$$

$$= 4\left[2\sin\theta - \frac{\theta}{2} - \frac{\sin 2\theta}{4}\right]_0^{\pi/2}$$

$$= 4\left(2 - \frac{\pi}{4}\right) = 8 - \pi$$

$$\boxed{A = 8 - \pi}$$

---

## 4.5 Famous Polar Areas

| Curve | Equation | Area |
|---|---|---|
| Circle | r = a | πa² |
| Cardioid | r = a(1 + cos θ) | 3πa²/2 |
| Rose (4 petals) | r = a sin 2θ | πa²/2 |
| Rose (3 petals) | r = a sin 3θ | πa²/4 |
| Lemniscate | r² = a² cos 2θ | a² |
| Limaçon | r = b + a cos θ | π(b² + a²/2) |
| Spiral (1 turn) | r = aθ | πa²(2π)²/3... |

---

## 4.6 Common Mistakes

```
  ⚠ PITFALL 1: Using full [0, 2π] when curve  
     doesn't exist everywhere
     → Check where r² ≥ 0 (e.g., lemniscate)
  
  ⚠ PITFALL 2: Forgetting to square r
     → Formula is ½∫r² dθ, NOT ½∫r dθ
  
  ⚠ PITFALL 3: Wrong limits for multi-petaled roses
     → One petal of r = sin(nθ): θ ∈ [0, π/n]
  
  ⚠ PITFALL 4: Counting petals wrong
     → r = sin(nθ): n petals if n odd,
                     2n petals if n even
```

---

## Summary Table

| Concept | Formula |
|---|---|
| Polar area | A = ½ ∫_α^β r² dθ |
| Between two curves | A = ½ ∫_α^β (r₂² − r₁²) dθ |
| Full circle | 0 ≤ θ ≤ 2π |
| Exploit symmetry | Multiply partial area by 2 or 4 |
| Rose r = sin nθ | n odd → n petals; n even → 2n petals |

---

## Quick Revision Questions

1. Derive the polar area formula A = ½ ∫ r² dθ from first principles.

2. Find the area enclosed by one petal of r = a cos 3θ.

3. Find the total area enclosed by the cardioid r = 1 − sin θ.

4. Find the area inside the circle r = 3sin θ.

5. Find the area that lies inside r = 1 + cos θ and outside r = 1.

6. For the lemniscate r² = 2 cos 2θ, find the total enclosed area and explain why we integrate only over [0, π/4].

---

[← Previous: Area in Parametric Form](03-area-parametric-form.md) | [Back to README](../README.md) | [Next: Disk Method →](../07-Applications-Volume/01-disk-method.md)
