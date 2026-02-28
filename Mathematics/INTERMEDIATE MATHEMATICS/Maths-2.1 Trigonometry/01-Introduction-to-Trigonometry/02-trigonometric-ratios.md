# Chapter 1.2: Trigonometric Ratios

## Overview

Trigonometric ratios are the foundation of trigonometry. They define the relationships between the sides of a right-angled triangle and its angles. This chapter introduces all six trigonometric ratios and their fundamental properties.

---

## 📐 The Right-Angled Triangle

Consider a right-angled triangle with:
- One angle of 90° (right angle)
- An acute angle θ (theta)
- Three sides with specific names relative to angle θ

```
                          B
                         /|
                        / |
                       /  |
              Hypotenuse  | Opposite
                (H)    /  | (O)
                      /   |
                     /θ   |
                    /_____|
                   A   C
                   Adjacent (A)
                   
    ∠ACB = 90° (Right angle)
    ∠BAC = θ (The angle we're considering)
```

### Side Naming Convention

| Side | Position Relative to θ | Symbol |
|------|------------------------|--------|
| **Opposite** | Across from angle θ | O |
| **Adjacent** | Next to angle θ (not hypotenuse) | A |
| **Hypotenuse** | Opposite the right angle (longest side) | H |

> **Important**: The "opposite" and "adjacent" sides change depending on which angle you're considering!

---

## 📊 The Six Trigonometric Ratios

### Primary Ratios (SOH-CAH-TOA)

The three primary ratios are **sine**, **cosine**, and **tangent**:

$$\sin \theta = \frac{\text{Opposite}}{\text{Hypotenuse}} = \frac{O}{H}$$

$$\cos \theta = \frac{\text{Adjacent}}{\text{Hypotenuse}} = \frac{A}{H}$$

$$\tan \theta = \frac{\text{Opposite}}{\text{Adjacent}} = \frac{O}{A}$$

### Memory Aid: **SOH-CAH-TOA**

```
┌─────────────────────────────────────────────────┐
│                 SOH - CAH - TOA                 │
├─────────────────────────────────────────────────┤
│  S-O-H          C-A-H          T-O-A            │
│    │              │              │              │
│  Sin=Opp/Hyp   Cos=Adj/Hyp   Tan=Opp/Adj       │
└─────────────────────────────────────────────────┘
```

### Reciprocal Ratios

The three reciprocal ratios are **cosecant**, **secant**, and **cotangent**:

$$\csc \theta = \frac{\text{Hypotenuse}}{\text{Opposite}} = \frac{H}{O} = \frac{1}{\sin \theta}$$

$$\sec \theta = \frac{\text{Hypotenuse}}{\text{Adjacent}} = \frac{H}{A} = \frac{1}{\cos \theta}$$

$$\cot \theta = \frac{\text{Adjacent}}{\text{Opposite}} = \frac{A}{O} = \frac{1}{\tan \theta}$$

### Complete Ratio Diagram

```
                    sin θ = O/H ←──────── csc θ = H/O
                         │                      │
                         │    RECIPROCALS       │
                         ↓                      ↓
    cos θ = A/H ←───────────────────────── sec θ = H/A
                         │                      │
                         │                      │
                         ↓                      ↓
    tan θ = O/A ←───────────────────────── cot θ = A/O
    
    Also: tan θ = sin θ / cos θ
          cot θ = cos θ / sin θ
```

---

## 🔗 Relationships Between Ratios

### Reciprocal Relationships

| Ratio | Reciprocal |
|-------|------------|
| sin θ | csc θ = 1/sin θ |
| cos θ | sec θ = 1/cos θ |
| tan θ | cot θ = 1/tan θ |

### Quotient Relationships

$$\tan \theta = \frac{\sin \theta}{\cos \theta}$$

$$\cot \theta = \frac{\cos \theta}{\sin \theta}$$

### Product Relationships

$$\sin \theta \cdot \csc \theta = 1$$
$$\cos \theta \cdot \sec \theta = 1$$
$$\tan \theta \cdot \cot \theta = 1$$

---

## 📐 Complementary Angle Relationships

For complementary angles (angles that sum to 90°):

```
                   B
                  /|
                 / |
                /  |
               /   |
              /    |
             / θ   | (90° - θ)
            /______|
           A       C
           
    ∠BAC = θ
    ∠ABC = 90° - θ
```

### Co-function Identities

| Function of θ | Equals Function of (90° - θ) |
|---------------|------------------------------|
| sin θ | cos(90° - θ) |
| cos θ | sin(90° - θ) |
| tan θ | cot(90° - θ) |
| cot θ | tan(90° - θ) |
| sec θ | csc(90° - θ) |
| csc θ | sec(90° - θ) |

> **Note**: The "co" in cosine, cotangent, and cosecant stands for "complementary"!

---

## 📏 Range of Trigonometric Ratios

For an acute angle θ (0° < θ < 90°):

| Ratio | Range |
|-------|-------|
| sin θ | 0 < sin θ < 1 |
| cos θ | 0 < cos θ < 1 |
| tan θ | 0 < tan θ < ∞ |
| csc θ | 1 < csc θ < ∞ |
| sec θ | 1 < sec θ < ∞ |
| cot θ | 0 < cot θ < ∞ |

---

## 🧮 Worked Examples

### Example 1: Finding All Six Ratios

Given a right triangle with opposite = 3, adjacent = 4, and hypotenuse = 5, find all six trigonometric ratios for angle θ.

```
              5
             /|
            / |
           /  | 3
          /θ  |
         /____|
           4
```

**Solution:**

| Ratio | Calculation | Value |
|-------|-------------|-------|
| sin θ | O/H = 3/5 | 0.6 |
| cos θ | A/H = 4/5 | 0.8 |
| tan θ | O/A = 3/4 | 0.75 |
| csc θ | H/O = 5/3 | 1.667 |
| sec θ | H/A = 5/4 | 1.25 |
| cot θ | A/O = 4/3 | 1.333 |

### Example 2: Finding Missing Side

If sin θ = 5/13 and the hypotenuse is 26 cm, find the opposite and adjacent sides.

**Solution:**
$$\sin \theta = \frac{\text{Opposite}}{\text{Hypotenuse}} = \frac{5}{13}$$

If H = 26:
$$\text{Opposite} = 26 \times \frac{5}{13} = 10 \text{ cm}$$

Using Pythagorean theorem:
$$\text{Adjacent}^2 = 26^2 - 10^2 = 676 - 100 = 576$$
$$\text{Adjacent} = 24 \text{ cm}$$

### Example 3: Proving a Relationship

Prove that tan θ = sin θ / cos θ

**Proof:**
$$\tan \theta = \frac{O}{A}$$

Multiply numerator and denominator by 1/H:
$$\tan \theta = \frac{O/H}{A/H} = \frac{\sin \theta}{\cos \theta}$$

### Example 4: Using Complementary Relationships

If sin 40° = 0.6428, find cos 50°.

**Solution:**
Since 40° + 50° = 90°:
$$\cos 50° = \sin(90° - 50°) = \sin 40° = 0.6428$$

### Example 5: Finding an Angle

In a right triangle, the opposite side is 7 units and the adjacent side is 7 units. Find angle θ.

**Solution:**
$$\tan \theta = \frac{O}{A} = \frac{7}{7} = 1$$

Therefore, θ = 45° (since tan 45° = 1)

---

## 🎯 Special Right Triangles

### The 45-45-90 Triangle (Isosceles Right Triangle)

```
         1
        /|
       / |
   √2 /  | 1
     /45°|
    /____|
       1
      45°
```

| Ratio | Value |
|-------|-------|
| sin 45° | 1/√2 = √2/2 |
| cos 45° | 1/√2 = √2/2 |
| tan 45° | 1 |

### The 30-60-90 Triangle

```
         2
        /|
       / |
      /  | 1
     /60°|
    /____|
      √3
     30°
```

| Angle | sin | cos | tan |
|-------|-----|-----|-----|
| 30° | 1/2 | √3/2 | 1/√3 |
| 60° | √3/2 | 1/2 | √3 |

---

## 🌍 Real-World Applications

### 1. Construction and Architecture
Determining roof pitch, calculating ladder angles, and designing ramps all use trigonometric ratios.

### 2. Navigation
Finding distances and directions using angles measured from landmarks.

### 3. Physics
Resolving forces into components, analyzing projectile motion, and studying wave phenomena.

### 4. Engineering
Designing mechanical systems, calculating stresses in structures, and analyzing circuits.

---

## 📋 Summary Table

| Ratio | Definition | Reciprocal | Range (acute angle) |
|-------|------------|------------|---------------------|
| sin θ | O/H | csc θ | (0, 1) |
| cos θ | A/H | sec θ | (0, 1) |
| tan θ | O/A | cot θ | (0, ∞) |
| csc θ | H/O | sin θ | (1, ∞) |
| sec θ | H/A | cos θ | (1, ∞) |
| cot θ | A/O | tan θ | (0, ∞) |

### Key Formulas

| Formula | Description |
|---------|-------------|
| tan θ = sin θ / cos θ | Quotient identity |
| cot θ = cos θ / sin θ | Quotient identity |
| sin θ · csc θ = 1 | Product identity |
| sin θ = cos(90° - θ) | Complementary identity |

---

## ❓ Quick Revision Questions

1. **In a right triangle, the hypotenuse is 17 and the opposite side is 8. Find all six trigonometric ratios.**

2. **If cos θ = 12/13, find sin θ and tan θ.** (Assume θ is acute)

3. **Prove that cot θ = cos θ / sin θ using the definitions of trigonometric ratios.**

4. **If sec θ = 5/3, find all other trigonometric ratios.**

5. **Why can't sin θ be greater than 1 for any angle in a right triangle?**

6. **If sin(90° - θ) = 0.8, what is cos θ?**

<details>
<summary>Click to see answers</summary>

1. Adjacent = √(17² - 8²) = √225 = 15  
   sin θ = 8/17, cos θ = 15/17, tan θ = 8/15  
   csc θ = 17/8, sec θ = 17/15, cot θ = 15/8

2. sin θ = √(1 - cos²θ) = √(1 - 144/169) = √(25/169) = **5/13**  
   tan θ = sin θ / cos θ = (5/13)/(12/13) = **5/12**

3. cot θ = A/O = (A/H)/(O/H) = cos θ / sin θ

4. cos θ = 3/5, sin θ = 4/5, tan θ = 4/3  
   csc θ = 5/4, cot θ = 3/4

5. In a right triangle, the hypotenuse is always the longest side. Since sin θ = Opposite/Hypotenuse, and Opposite < Hypotenuse, sin θ must be less than 1.

6. sin(90° - θ) = cos θ, so **cos θ = 0.8**

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 1.1 Angle Measurement](01-angle-measurement.md) | [Unit 1 Index](README.md) | [1.3 Standard Angles →](03-standard-angles.md) |

---

[← Back to Main Index](../README.md)
