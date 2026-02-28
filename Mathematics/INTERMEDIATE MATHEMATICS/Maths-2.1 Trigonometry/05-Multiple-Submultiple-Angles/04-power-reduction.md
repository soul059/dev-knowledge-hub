# Chapter 5.4: Power Reduction Formulas

## Overview

Power reduction formulas allow us to express **powers** of sine and cosine in terms of **first powers** of cosine of multiple angles. These formulas are essential for integration and simplifying expressions involving sin²x, cos²x, sin⁴x, cos⁴x, etc.

---

## 📐 The Basic Power Reduction Formulas

### Squared Functions

$$\boxed{\sin^2 A = \frac{1 - \cos 2A}{2}}$$

$$\boxed{\cos^2 A = \frac{1 + \cos 2A}{2}}$$

$$\boxed{\sin A \cos A = \frac{\sin 2A}{2}}$$

### Alternative Forms

$$\sin^2 A = \frac{1}{2}(1 - \cos 2A)$$

$$\cos^2 A = \frac{1}{2}(1 + \cos 2A)$$

---

## 🔍 Derivations

### Deriving sin²A Formula

From the double angle formula:
$$\cos 2A = 1 - 2\sin^2 A$$

Solving for sin²A:
$$2\sin^2 A = 1 - \cos 2A$$
$$\sin^2 A = \frac{1 - \cos 2A}{2}$$

### Deriving cos²A Formula

From the double angle formula:
$$\cos 2A = 2\cos^2 A - 1$$

Solving for cos²A:
$$2\cos^2 A = 1 + \cos 2A$$
$$\cos^2 A = \frac{1 + \cos 2A}{2}$$

### Deriving sin A cos A Formula

From the double angle formula:
$$\sin 2A = 2\sin A \cos A$$

Solving for sin A cos A:
$$\sin A \cos A = \frac{\sin 2A}{2}$$

---

## 📊 Formula Reference Chart

```
    ┌─────────────────────────────────────────────────────────────────┐
    │                   POWER REDUCTION FORMULAS                      │
    ├─────────────────────────────────────────────────────────────────┤
    │                                                                 │
    │  ┌─────────────────────────────────────────────────────────┐   │
    │  │                   BASIC FORMS                           │   │
    │  ├─────────────────────────────────────────────────────────┤   │
    │  │                                                         │   │
    │  │  sin²A = (1 - cos 2A)/2                                │   │
    │  │            ↑                                            │   │
    │  │         MINUS (think: sin starts at 0)                  │   │
    │  │                                                         │   │
    │  │  cos²A = (1 + cos 2A)/2                                │   │
    │  │            ↑                                            │   │
    │  │         PLUS (think: cos starts at 1)                   │   │
    │  │                                                         │   │
    │  │  sin A cos A = sin 2A / 2                              │   │
    │  │                                                         │   │
    │  └─────────────────────────────────────────────────────────┘   │
    │                                                                 │
    └─────────────────────────────────────────────────────────────────┘
```

---

## 📐 Higher Power Formulas

### Fourth Power

$$\boxed{\sin^4 A = \frac{3 - 4\cos 2A + \cos 4A}{8}}$$

$$\boxed{\cos^4 A = \frac{3 + 4\cos 2A + \cos 4A}{8}}$$

### Derivation of sin⁴A

$$\sin^4 A = (\sin^2 A)^2 = \left(\frac{1 - \cos 2A}{2}\right)^2$$
$$= \frac{1 - 2\cos 2A + \cos^2 2A}{4}$$

Now apply power reduction to cos²2A:
$$\cos^2 2A = \frac{1 + \cos 4A}{2}$$

$$\sin^4 A = \frac{1 - 2\cos 2A + \frac{1 + \cos 4A}{2}}{4}$$
$$= \frac{2 - 4\cos 2A + 1 + \cos 4A}{8}$$
$$= \frac{3 - 4\cos 2A + \cos 4A}{8}$$

### Derivation of cos⁴A

$$\cos^4 A = (\cos^2 A)^2 = \left(\frac{1 + \cos 2A}{2}\right)^2$$
$$= \frac{1 + 2\cos 2A + \cos^2 2A}{4}$$
$$= \frac{1 + 2\cos 2A + \frac{1 + \cos 4A}{2}}{4}$$
$$= \frac{2 + 4\cos 2A + 1 + \cos 4A}{8}$$
$$= \frac{3 + 4\cos 2A + \cos 4A}{8}$$

---

## 🧠 Memory Techniques

### For sin²A and cos²A

```
    Think of what happens at A = 0:
    
    sin²(0) = 0     →  Formula should give 0  →  (1 - cos 0)/2 = 0  ✓
    cos²(0) = 1     →  Formula should give 1  →  (1 + cos 0)/2 = 1  ✓
    
    So:
    sin² → has MINUS (to make it zero at A = 0)
    cos² → has PLUS (to make it one at A = 0)
```

### Pattern for Higher Powers

| Power | Pattern |
|-------|---------|
| sin²A, cos²A | Involves cos 2A |
| sin⁴A, cos⁴A | Involves cos 2A and cos 4A |
| sin⁶A, cos⁶A | Involves cos 2A, cos 4A, and cos 6A |

---

## 🧮 Worked Examples

### Example 1: Simplify sin²30°

**Solution:**
$$\sin^2 30° = \frac{1 - \cos 60°}{2} = \frac{1 - \frac{1}{2}}{2} = \frac{\frac{1}{2}}{2} = \boxed{\frac{1}{4}}$$

**Verification:** sin 30° = 1/2, so sin²30° = 1/4 ✓

### Example 2: Simplify cos²45°

**Solution:**
$$\cos^2 45° = \frac{1 + \cos 90°}{2} = \frac{1 + 0}{2} = \boxed{\frac{1}{2}}$$

**Verification:** cos 45° = √2/2, so cos²45° = 1/2 ✓

### Example 3: Integrate sin²x

**Solution:**
$$\int \sin^2 x \, dx = \int \frac{1 - \cos 2x}{2} \, dx$$
$$= \frac{1}{2}\int (1 - \cos 2x) \, dx$$
$$= \frac{1}{2}\left(x - \frac{\sin 2x}{2}\right) + C$$
$$= \boxed{\frac{x}{2} - \frac{\sin 2x}{4} + C}$$

### Example 4: Integrate cos²x

**Solution:**
$$\int \cos^2 x \, dx = \int \frac{1 + \cos 2x}{2} \, dx$$
$$= \frac{1}{2}\int (1 + \cos 2x) \, dx$$
$$= \frac{1}{2}\left(x + \frac{\sin 2x}{2}\right) + C$$
$$= \boxed{\frac{x}{2} + \frac{\sin 2x}{4} + C}$$

### Example 5: Simplify sin⁴x + cos⁴x

**Solution:**
Using the formulas:
$$\sin^4 x + \cos^4 x = \frac{3 - 4\cos 2x + \cos 4x}{8} + \frac{3 + 4\cos 2x + \cos 4x}{8}$$
$$= \frac{6 + 2\cos 4x}{8}$$
$$= \frac{3 + \cos 4x}{4}$$

**Alternative method:**
$$\sin^4 x + \cos^4 x = (\sin^2 x + \cos^2 x)^2 - 2\sin^2 x \cos^2 x$$
$$= 1 - 2\left(\frac{\sin 2x}{2}\right)^2$$
$$= 1 - \frac{\sin^2 2x}{2}$$
$$= 1 - \frac{1 - \cos 4x}{4}$$
$$= \boxed{\frac{3 + \cos 4x}{4}}$$

### Example 6: Integrate sin⁴x

**Solution:**
$$\int \sin^4 x \, dx = \int \frac{3 - 4\cos 2x + \cos 4x}{8} \, dx$$
$$= \frac{1}{8}\int (3 - 4\cos 2x + \cos 4x) \, dx$$
$$= \frac{1}{8}\left(3x - 2\sin 2x + \frac{\sin 4x}{4}\right) + C$$
$$= \boxed{\frac{3x}{8} - \frac{\sin 2x}{4} + \frac{\sin 4x}{32} + C}$$

---

## 📊 Useful Identities from Power Reduction

### Sum of Squared Functions

$$\sin^2 A + \cos^2 A = 1$$

### Difference of Squared Functions

$$\cos^2 A - \sin^2 A = \cos 2A$$

### Product Forms

$$\sin^2 A \cos^2 A = \frac{\sin^2 2A}{4} = \frac{1 - \cos 4A}{8}$$

---

## 📈 Graphical Interpretation

### sin²x Graph

```
    y
    │
  1 ┤    ___       ___       ___
    │   /   \     /   \     /   \
    │  /     \   /     \   /     \
0.5 ┤ /       \ /       \ /       
    │/         X         X
  0 ┼─────────────────────────────── x
    0    π/2    π    3π/2   2π
    
    sin²x oscillates between 0 and 1
    Average value = 1/2
    Period = π (half of sin x)
```

### cos²x Graph

```
    y
    │
  1 ┤\         /\         /\
    │ \       /  \       /  \
    │  \     /    \     /    \
0.5 ┤   \   /      \   /      \
    │    \ /        \ /        \
  0 ┼─────X──────────X────────── x
    0    π/2    π    3π/2   2π
    
    cos²x also oscillates between 0 and 1
    Average value = 1/2
    Period = π
```

---

## 🌍 Applications

### 1. RMS (Root Mean Square) Voltage/Current

In AC circuits:
$$V_{rms} = \sqrt{\frac{1}{T}\int_0^T V_0^2\sin^2(\omega t) \, dt} = \frac{V_0}{\sqrt{2}}$$

### 2. Light Intensity (Malus's Law)

$$I = I_0 \cos^2\theta$$

Intensity of polarized light after passing through a polarizer.

### 3. Power in AC Circuits

$$P = V_0 I_0 \sin^2(\omega t) = \frac{V_0 I_0}{2}(1 - \cos 2\omega t)$$

### 4. Fourier Analysis

Expressing signals in terms of fundamental frequencies.

---

## 📋 Summary Table

### Basic Power Reduction Formulas

| Expression | Reduced Form |
|------------|--------------|
| sin²A | (1 - cos 2A)/2 |
| cos²A | (1 + cos 2A)/2 |
| sin A cos A | sin 2A/2 |

### Fourth Power Formulas

| Expression | Reduced Form |
|------------|--------------|
| sin⁴A | (3 - 4cos 2A + cos 4A)/8 |
| cos⁴A | (3 + 4cos 2A + cos 4A)/8 |
| sin²A cos²A | (1 - cos 4A)/8 |

### Integration Results

| Integral | Result |
|----------|--------|
| ∫sin²x dx | x/2 - sin 2x/4 + C |
| ∫cos²x dx | x/2 + sin 2x/4 + C |
| ∫sin⁴x dx | 3x/8 - sin 2x/4 + sin 4x/32 + C |
| ∫cos⁴x dx | 3x/8 + sin 2x/4 + sin 4x/32 + C |

---

## ❓ Quick Revision Questions

1. **Express sin²(3x) in terms of cos 6x.**

2. **Simplify: 1 - 2sin²A (express as a single term).**

3. **Evaluate ∫₀^π sin²x dx.**

4. **Prove that sin⁴A - cos⁴A = -cos 2A.**

5. **Express sin⁶A in terms of cosines of multiple angles. (Hint: sin⁶A = sin²A · sin⁴A)**

6. **Why are power reduction formulas essential for integration?**

<details>
<summary>Click to see answers</summary>

1. sin²(3x) = (1 - cos 6x)/2  
   = **(1 - cos 6x)/2** or **½ - ½cos 6x**

2. 1 - 2sin²A = **cos 2A**  
   (This is the double angle formula directly!)

3. ∫₀^π sin²x dx = ∫₀^π (1 - cos 2x)/2 dx  
   = [x/2 - sin 2x/4]₀^π  
   = (π/2 - 0) - (0 - 0)  
   = **π/2**

4. sin⁴A - cos⁴A = (sin²A + cos²A)(sin²A - cos²A)  
   = 1 · (sin²A - cos²A)  
   = -(cos²A - sin²A)  
   = **-cos 2A** ✓

5. sin⁶A = (sin²A)³  
   = [(1 - cos 2A)/2]³  
   = (1/8)(1 - cos 2A)³  
   = (1/8)(1 - 3cos 2A + 3cos²2A - cos³2A)  
   
   Using cos²2A = (1 + cos 4A)/2 and cos³2A = (3cos 2A + cos 6A)/4:
   = (1/8)[1 - 3cos 2A + 3(1 + cos 4A)/2 - (3cos 2A + cos 6A)/4]  
   = (1/8)[1 - 3cos 2A + 3/2 + 3cos 4A/2 - 3cos 2A/4 - cos 6A/4]  
   = **(10 - 15cos 2A + 6cos 4A - cos 6A)/32**

6. Integrals like ∫sin²x dx or ∫cos⁴x dx cannot be solved directly. Power reduction converts these to integrals of cosines of multiple angles, which are straightforward to integrate:  
   ∫cos(nx) dx = sin(nx)/n + C  
   Without power reduction, we would need much more complex techniques.

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 5.3 Half Angle Formulas](03-half-angle.md) | [Unit 5 Index](README.md) | [Unit 6: Trigonometric Equations →](../06-Trigonometric-Equations/README.md) |

---

[← Back to Main Index](../README.md)
