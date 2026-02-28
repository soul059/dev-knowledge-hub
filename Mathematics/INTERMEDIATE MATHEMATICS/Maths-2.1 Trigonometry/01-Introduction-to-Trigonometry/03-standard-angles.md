# Chapter 1.3: Ratios of Standard Angles

## Overview

Certain angles appear so frequently in mathematics and science that their trigonometric values should be memorized. This chapter covers the exact values for the standard angles: **0°, 30°, 45°, 60°, and 90°** and provides techniques to remember them.

---

## 📐 Deriving Standard Angle Values

### The 45° Angle (π/4 radians)

Consider an isosceles right triangle with legs of length 1:

```
                    B
                    |\
                    | \
                    |  \
                 1  |   \ √2
                    |    \
                    |45°  \
                    |______\
                   A   1    C
                        45°
                        
    By Pythagorean Theorem:
    BC² = AB² + AC² = 1² + 1² = 2
    BC = √2
```

**Calculations:**
$$\sin 45° = \frac{AB}{BC} = \frac{1}{\sqrt{2}} = \frac{\sqrt{2}}{2}$$

$$\cos 45° = \frac{AC}{BC} = \frac{1}{\sqrt{2}} = \frac{\sqrt{2}}{2}$$

$$\tan 45° = \frac{AB}{AC} = \frac{1}{1} = 1$$

---

### The 30° and 60° Angles (π/6 and π/3 radians)

Start with an equilateral triangle with side length 2, then draw an altitude:

```
                    A
                   /\
                  /  \
                 /    \
              2 /      \ 2
               /   |    \
              /    |     \
             /  30°|      \
            /______|_______\
           B   1   D   1    C
                   
    Equilateral triangle ABC with altitude AD
    ∠BAC = 60°, after bisection ∠BAD = 30°
    BD = DC = 1 (altitude bisects base)
    AD = √(2² - 1²) = √3
```

**For 30° (at vertex A):**
$$\sin 30° = \frac{BD}{AB} = \frac{1}{2}$$

$$\cos 30° = \frac{AD}{AB} = \frac{\sqrt{3}}{2}$$

$$\tan 30° = \frac{BD}{AD} = \frac{1}{\sqrt{3}} = \frac{\sqrt{3}}{3}$$

**For 60° (at vertex B):**
$$\sin 60° = \frac{AD}{AB} = \frac{\sqrt{3}}{2}$$

$$\cos 60° = \frac{BD}{AB} = \frac{1}{2}$$

$$\tan 60° = \frac{AD}{BD} = \frac{\sqrt{3}}{1} = \sqrt{3}$$

---

### The 0° and 90° Angles (Limiting Cases)

Consider what happens as angle θ approaches 0° and 90°:

```
    As θ → 0°:                    As θ → 90°:
    
         ___θ→0°___                    |
        /   →  H→A                     | O→H
       /_________                   ___|
         A                             (A→0)
         
    Opposite → 0                   Adjacent → 0
    Adjacent → Hypotenuse          Opposite → Hypotenuse
```

**At 0°:**
- sin 0° = 0 (no opposite side)
- cos 0° = 1 (adjacent = hypotenuse)
- tan 0° = 0

**At 90°:**
- sin 90° = 1 (opposite = hypotenuse)
- cos 90° = 0 (no adjacent side)
- tan 90° = undefined (division by zero)

---

## 📊 Complete Standard Angle Table

### Primary Trigonometric Ratios

| Angle | 0° | 30° | 45° | 60° | 90° |
|-------|-----|------|------|------|------|
| **Radians** | 0 | π/6 | π/4 | π/3 | π/2 |
| **sin θ** | 0 | 1/2 | √2/2 | √3/2 | 1 |
| **cos θ** | 1 | √3/2 | √2/2 | 1/2 | 0 |
| **tan θ** | 0 | √3/3 | 1 | √3 | ∞ |

### Reciprocal Trigonometric Ratios

| Angle | 0° | 30° | 45° | 60° | 90° |
|-------|-----|------|------|------|------|
| **csc θ** | ∞ | 2 | √2 | 2√3/3 | 1 |
| **sec θ** | 1 | 2√3/3 | √2 | 2 | ∞ |
| **cot θ** | ∞ | √3 | 1 | √3/3 | 0 |

---

## 🧠 Memory Techniques

### Method 1: The "Square Root Pattern"

For sine values from 0° to 90°:

```
    sin 0°  = √0/2 = 0
    sin 30° = √1/2 = 1/2
    sin 45° = √2/2
    sin 60° = √3/2
    sin 90° = √4/2 = 1
    
    Pattern: √0, √1, √2, √3, √4 all divided by 2
```

For cosine, the pattern is **reversed** (read bottom to top).

### Method 2: The Hand Trick

```
                                    90° (4)
                                     |
                    60° (3)          |          
                         \           |
              45° (2)     \          |
                   \       \         |
        30° (1)     \       \        |
             \       \       \       |
    0° (0)____\____________________|
              
    Each finger represents an angle:
    Thumb = 0°, Index = 30°, Middle = 45°, 
    Ring = 60°, Pinky = 90°
    
    For sin: √(finger number)/2
    For cos: √(4 - finger number)/2
```

### Method 3: Visual Pattern Table

```
    ┌────────────────────────────────────────────┐
    │         The Pattern for sin θ              │
    ├────────────────────────────────────────────┤
    │  0°     30°      45°      60°      90°     │
    │   ↓      ↓        ↓        ↓        ↓      │
    │  √0     √1       √2       √3       √4      │
    │  ──     ──       ──       ──       ──      │
    │   2      2        2        2        2      │
    │   ↓      ↓        ↓        ↓        ↓      │
    │   0     1/2    √2/2     √3/2       1       │
    └────────────────────────────────────────────┘
    
    For cos θ: Use the same values in REVERSE order!
```

---

## 📈 Graphical Representation

### Sine Values Visualization

```
    sin θ
      1 ─┼───────────────────────────●  90°
        │                        ╱
    0.87─┼─────────────────────●     60°
        │                   ╱
    0.71─┼────────────────●          45°
        │              ╱
     0.5─┼───────────●               30°
        │         ╱
        │      ╱
      0 ●────┴─────────────────────────→ θ
        0°   30°   45°   60°   90°
```

### Cosine Values Visualization

```
    cos θ
      1 ●                               0°
        │\
    0.87─┼──●                           30°
        │    \
    0.71─┼─────●                        45°
        │       \
     0.5─┼────────●                     60°
        │          \
        │            \
      0 ─┼─────────────●───────────→ θ  90°
        0°   30°   45°   60°   90°
```

### Tangent Values Visualization

```
    tan θ
      ∞ ┼                          │   90°
        │                          │
      2 ┼                          │
        │                       ╱  │
    √3 ─┼─────────────────────●    │   60°
        │                  ╱       │
      1 ─┼───────────────●─────────┤   45°
        │             ╱            │
    1/√3┼───────────●              │   30°
        │        ╱                 │
      0 ●───────┴──────────────────┴→ θ
        0°   30°   45°   60°   90°
```

---

## 🧮 Worked Examples

### Example 1: Exact Value Calculations

Evaluate: $\sin 60° \cos 30° + \cos 60° \sin 30°$

**Solution:**
$$= \frac{\sqrt{3}}{2} \cdot \frac{\sqrt{3}}{2} + \frac{1}{2} \cdot \frac{1}{2}$$
$$= \frac{3}{4} + \frac{1}{4} = \frac{4}{4} = 1$$

### Example 2: Simplifying Expressions

Simplify: $\frac{\tan 45° + \tan 30°}{1 - \tan 45° \tan 30°}$

**Solution:**
$$= \frac{1 + \frac{1}{\sqrt{3}}}{1 - 1 \cdot \frac{1}{\sqrt{3}}}$$
$$= \frac{\frac{\sqrt{3}+1}{\sqrt{3}}}{\frac{\sqrt{3}-1}{\sqrt{3}}}$$
$$= \frac{\sqrt{3}+1}{\sqrt{3}-1}$$

Rationalizing:
$$= \frac{(\sqrt{3}+1)^2}{(\sqrt{3}-1)(\sqrt{3}+1)} = \frac{3 + 2\sqrt{3} + 1}{3-1} = \frac{4 + 2\sqrt{3}}{2} = 2 + \sqrt{3}$$

### Example 3: Verification

Verify that $\sin^2 30° + \cos^2 30° = 1$

**Solution:**
$$\sin^2 30° + \cos^2 30° = \left(\frac{1}{2}\right)^2 + \left(\frac{\sqrt{3}}{2}\right)^2$$
$$= \frac{1}{4} + \frac{3}{4} = \frac{4}{4} = 1 \checkmark$$

### Example 4: Finding Unknown Angles

If $\cos \theta = \frac{\sqrt{3}}{2}$ and θ is acute, find θ and all trigonometric ratios.

**Solution:**
From the standard angle table, $\cos 30° = \frac{\sqrt{3}}{2}$

Therefore, **θ = 30°**

All ratios:
- sin 30° = 1/2
- cos 30° = √3/2
- tan 30° = 1/√3 = √3/3
- csc 30° = 2
- sec 30° = 2/√3 = 2√3/3
- cot 30° = √3

### Example 5: Solving Triangles

In a right triangle, one acute angle is 60° and the hypotenuse is 10 cm. Find all sides.

```
                 C
                 |\
                 | \
      Opposite   |  \ 10 (Hypotenuse)
                 |   \
                 |60° \
                 |_____\
                A   B
                Adjacent
```

**Solution:**
$$\sin 60° = \frac{\text{Opposite}}{10} \Rightarrow \text{Opposite} = 10 \times \frac{\sqrt{3}}{2} = 5\sqrt{3} \text{ cm}$$

$$\cos 60° = \frac{\text{Adjacent}}{10} \Rightarrow \text{Adjacent} = 10 \times \frac{1}{2} = 5 \text{ cm}$$

---

## 🔗 Relationships Between Standard Angles

### Complementary Angle Pairs

Since 30° + 60° = 90° and 0° + 90° = 90°:

| Relationship | Example |
|--------------|---------|
| sin 30° = cos 60° | 1/2 = 1/2 ✓ |
| sin 60° = cos 30° | √3/2 = √3/2 ✓ |
| tan 30° = cot 60° | 1/√3 = 1/√3 ✓ |
| sin 0° = cos 90° | 0 = 0 ✓ |

### Symmetry at 45°

At θ = 45°, sine and cosine are equal:
$$\sin 45° = \cos 45° = \frac{\sqrt{2}}{2}$$

This is because 45° is exactly halfway between 0° and 90°.

---

## 🌍 Real-World Applications

### 1. Architecture
- 30-60-90 triangles appear in roof designs
- 45° angles are common in corner braces

### 2. Engineering
- 60° is used in hexagonal bolt heads
- 45° appears in scissor lifts and linkages

### 3. Nature
- Snowflakes have 60° symmetry
- Honeycomb cells use 120° angles (2 × 60°)

### 4. Sports
- Optimal projectile angle is 45° (ignoring air resistance)
- Pool/billiards players use angle calculations

---

## 📋 Summary Table

### Quick Reference - Exact Values

| θ | sin θ | cos θ | tan θ |
|---|-------|-------|-------|
| 0° | 0 | 1 | 0 |
| 30° | 1/2 | √3/2 | √3/3 |
| 45° | √2/2 | √2/2 | 1 |
| 60° | √3/2 | 1/2 | √3 |
| 90° | 1 | 0 | undefined |

### Decimal Approximations

| θ | sin θ | cos θ | tan θ |
|---|-------|-------|-------|
| 0° | 0 | 1 | 0 |
| 30° | 0.5 | 0.866 | 0.577 |
| 45° | 0.707 | 0.707 | 1 |
| 60° | 0.866 | 0.5 | 1.732 |
| 90° | 1 | 0 | ∞ |

### Key Patterns

| Pattern | Description |
|---------|-------------|
| sin increases | 0 → 1 as θ goes 0° → 90° |
| cos decreases | 1 → 0 as θ goes 0° → 90° |
| tan increases | 0 → ∞ as θ goes 0° → 90° |
| sin θ = cos(90° - θ) | Complementary relationship |

---

## ❓ Quick Revision Questions

1. **Write the exact value of sin 60° × cos 60° without a calculator.**

2. **If tan θ = √3, find θ (acute) and all other trigonometric ratios.**

3. **Evaluate: $\frac{\sin^2 45°}{\cos 0°} + \frac{\cos^2 45°}{\sin 90°}$**

4. **Prove that $\tan 30° \times \tan 60° = 1$**

5. **At what angle does sin θ = cos θ? Explain why.**

6. **A right triangle has angles 30°, 60°, and 90°. If the side opposite 30° is 4 cm, find the hypotenuse and the side opposite 60°.**

<details>
<summary>Click to see answers</summary>

1. sin 60° × cos 60° = (√3/2) × (1/2) = **√3/4**

2. tan θ = √3 means **θ = 60°**  
   sin 60° = √3/2, cos 60° = 1/2  
   cot 60° = 1/√3, sec 60° = 2, csc 60° = 2/√3

3. = (√2/2)²/1 + (√2/2)²/1 = 1/2 + 1/2 = **1**

4. tan 30° × tan 60° = (1/√3) × √3 = 1 ✓  
   (Also: tan θ × tan(90° - θ) = tan θ × cot θ = 1)

5. sin θ = cos θ at **θ = 45°**  
   Because: sin θ = cos(90° - θ), so θ = 90° - θ, giving 2θ = 90°, θ = 45°

6. sin 30° = opposite/hypotenuse = 4/H  
   1/2 = 4/H → **H = 8 cm**  
   Side opposite 60° = H × sin 60° = 8 × √3/2 = **4√3 cm**

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 1.2 Trigonometric Ratios](02-trigonometric-ratios.md) | [Unit 1 Index](README.md) | [1.4 Trigonometric Tables →](04-trigonometric-tables.md) |

---

[← Back to Main Index](../README.md)
