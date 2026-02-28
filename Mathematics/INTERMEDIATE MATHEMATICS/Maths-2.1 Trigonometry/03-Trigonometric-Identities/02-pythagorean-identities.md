# Chapter 3.2: Pythagorean Identities

## Overview

The Pythagorean identities are among the most important relationships in trigonometry. Derived from the Pythagorean theorem and the unit circle, these identities connect squares of trigonometric functions and are essential for simplifying expressions, solving equations, and proving other identities.

---

## 📐 The Three Pythagorean Identities

### Primary Identity

$$\boxed{\sin^2\theta + \cos^2\theta = 1}$$

### Derived Identities

$$\boxed{1 + \tan^2\theta = \sec^2\theta}$$

$$\boxed{1 + \cot^2\theta = \csc^2\theta}$$

---

## 🔍 Derivation from First Principles

### Identity 1: sin²θ + cos²θ = 1

**From the Unit Circle:**

```
                    y
                    │
                    │    P(cos θ, sin θ)
                    │   ●
                    │  /│
                    │ / │
                  1│/  │ sin θ
                    │/θ  │
    ────────────────●────┴────────── x
                    │  cos θ
                    │
                    
    Since P is on the unit circle: x² + y² = 1
    
    Therefore: cos²θ + sin²θ = 1
```

**From the Pythagorean Theorem:**

```
                    C
                    │\
                    │ \
                  a │  \ c (hypotenuse)
                    │   \
                    │  θ \
                    │_____\
                   A   b   B
                   
    Pythagorean Theorem: a² + b² = c²
    
    Divide by c²:  a²/c² + b²/c² = 1
    
                   sin²θ + cos²θ = 1  ✓
```

### Identity 2: 1 + tan²θ = sec²θ

**Derivation:**

Starting with sin²θ + cos²θ = 1:

$$\frac{\sin^2\theta}{\cos^2\theta} + \frac{\cos^2\theta}{\cos^2\theta} = \frac{1}{\cos^2\theta}$$

$$\tan^2\theta + 1 = \sec^2\theta$$

$$\boxed{1 + \tan^2\theta = \sec^2\theta}$$

### Identity 3: 1 + cot²θ = csc²θ

**Derivation:**

Starting with sin²θ + cos²θ = 1:

$$\frac{\sin^2\theta}{\sin^2\theta} + \frac{\cos^2\theta}{\sin^2\theta} = \frac{1}{\sin^2\theta}$$

$$1 + \cot^2\theta = \csc^2\theta$$

$$\boxed{1 + \cot^2\theta = \csc^2\theta}$$

---

## 📊 All Equivalent Forms

### Forms of sin²θ + cos²θ = 1

| Rearrangement | Form |
|---------------|------|
| Solve for sin²θ | sin²θ = 1 - cos²θ |
| Solve for cos²θ | cos²θ = 1 - sin²θ |
| Factor (difference of squares) | (sin θ + cos θ)(sin θ - cos θ) = sin²θ - cos²θ |

### Forms of 1 + tan²θ = sec²θ

| Rearrangement | Form |
|---------------|------|
| Solve for tan²θ | tan²θ = sec²θ - 1 |
| Alternative form | sec²θ - tan²θ = 1 |
| Factored | (sec θ + tan θ)(sec θ - tan θ) = 1 |

### Forms of 1 + cot²θ = csc²θ

| Rearrangement | Form |
|---------------|------|
| Solve for cot²θ | cot²θ = csc²θ - 1 |
| Alternative form | csc²θ - cot²θ = 1 |
| Factored | (csc θ + cot θ)(csc θ - cot θ) = 1 |

---

## 🎯 Visual Memory Aid

```
    ┌────────────────────────────────────────────────────────────┐
    │          PYTHAGOREAN IDENTITY TRIANGLE                     │
    ├────────────────────────────────────────────────────────────┤
    │                                                            │
    │                         1                                  │
    │                        /│                                  │
    │                       / │                                  │
    │             sec θ    /  │  csc θ                           │
    │                     /   │                                  │
    │                    /    │                                  │
    │                   /     │                                  │
    │                  ●──────●                                  │
    │               tan θ   cot θ                                │
    │                                                            │
    │   sin²θ + cos²θ = 1     ← Primary (on unit circle)        │
    │   tan²θ + 1 = sec²θ     ← Divide by cos²θ                 │
    │   1 + cot²θ = csc²θ     ← Divide by sin²θ                 │
    │                                                            │
    └────────────────────────────────────────────────────────────┘
```

### Relationship Pattern

```
    Identity 1:    sin²θ  +  cos²θ  =  1
                     ↓         ↓        ↓
    ÷ cos²θ:       tan²θ  +    1    =  sec²θ
    
    Identity 1:    sin²θ  +  cos²θ  =  1
                     ↓         ↓        ↓
    ÷ sin²θ:         1    +  cot²θ  =  csc²θ
```

---

## 🧮 Worked Examples

### Example 1: Basic Simplification

Simplify: sin²θ + cos²θ + tan²θ

**Solution:**
$$= (sin^2\theta + cos^2\theta) + tan^2\theta$$
$$= 1 + tan^2\theta$$
$$= \boxed{sec^2\theta}$$

### Example 2: Finding Function Values

If sin θ = 5/13 and θ is in Quadrant II, find cos θ and tan θ.

**Solution:**

Using sin²θ + cos²θ = 1:
$$\left(\frac{5}{13}\right)^2 + \cos^2\theta = 1$$
$$\frac{25}{169} + \cos^2\theta = 1$$
$$\cos^2\theta = \frac{144}{169}$$
$$\cos\theta = \pm\frac{12}{13}$$

In Quadrant II, cos θ is negative:
$$\cos\theta = -\frac{12}{13}$$

$$\tan\theta = \frac{\sin\theta}{\cos\theta} = \frac{5/13}{-12/13} = -\frac{5}{12}$$

### Example 3: Simplify Complex Expression

Simplify: $\frac{1 - \sin^2\theta}{\cos\theta}$

**Solution:**
$$= \frac{cos^2\theta}{\cos\theta}$$  [since 1 - sin²θ = cos²θ]
$$= \boxed{\cos\theta}$$

### Example 4: Prove an Identity

Prove: $\sec^4\theta - \tan^4\theta = \sec^2\theta + \tan^2\theta$

**Proof:**
$$LHS = \sec^4\theta - \tan^4\theta$$
$$= (\sec^2\theta)^2 - (\tan^2\theta)^2$$
$$= (\sec^2\theta + \tan^2\theta)(\sec^2\theta - \tan^2\theta)$$
$$= (\sec^2\theta + \tan^2\theta)(1)$$  [since sec²θ - tan²θ = 1]
$$= \sec^2\theta + \tan^2\theta = RHS \quad \checkmark$$

### Example 5: Converting Expressions

Express sec θ + tan θ in terms of sin θ and cos θ, then simplify sec θ - tan θ.

**Solution:**

Part 1:
$$\sec\theta + \tan\theta = \frac{1}{\cos\theta} + \frac{\sin\theta}{\cos\theta} = \frac{1 + \sin\theta}{\cos\theta}$$

Part 2: We know that:
$$(\sec\theta + \tan\theta)(\sec\theta - \tan\theta) = \sec^2\theta - \tan^2\theta = 1$$

Therefore:
$$\sec\theta - \tan\theta = \frac{1}{\sec\theta + \tan\theta} = \frac{\cos\theta}{1 + \sin\theta}$$

### Example 6: Finding All Values

If sec θ = -3 and π < θ < 3π/2, find all six trigonometric values.

**Solution:**

Given: sec θ = -3, θ is in Q III

$$\cos\theta = \frac{1}{\sec\theta} = -\frac{1}{3}$$

Using sin²θ + cos²θ = 1:
$$\sin^2\theta = 1 - \frac{1}{9} = \frac{8}{9}$$
$$\sin\theta = -\frac{2\sqrt{2}}{3}$$ (negative in Q III)

| Function | Value |
|----------|-------|
| sin θ | -2√2/3 |
| cos θ | -1/3 |
| tan θ | sin θ/cos θ = 2√2 |
| csc θ | -3/(2√2) = -3√2/4 |
| sec θ | -3 |
| cot θ | 1/(2√2) = √2/4 |

---

## 📈 Graphical Interpretation

### The Identity sin²θ + cos²θ = 1

```
    For any angle θ, the point (cos θ, sin θ) lies on the unit circle.
    
          y = sin θ
            │
          1 ┼─────────────●──────────────
            │            ╱│╲
            │           ╱ │ ╲
            │          ╱  │  ╲
            │         ╱   │   ╲
            │        ╱    │    ╲
            │       ╱     │     ╲
            │      ╱      │      ╲
    ────────┼─────●───────┼───────●──────  x = cos θ
           -1     │       0       │      1
            │      ╲      │      ╱
            │       ╲     │     ╱
            │        ╲    │    ╱
            │         ╲   │   ╱
         -1 ┼──────────╲──┼──╱───────────
                        ╲│╱
                         ●
                         
    Every point on this circle satisfies x² + y² = 1
    i.e., cos²θ + sin²θ = 1
```

### Relationship Between Identities

```
    sin²θ + cos²θ = 1
         ╱           ╲
        ╱             ╲
       ÷cos²θ         ÷sin²θ
       ╱               ╲
      ╱                 ╲
    1 + tan²θ = sec²θ   1 + cot²θ = csc²θ
```

---

## 🔢 Common Mistakes to Avoid

### Mistake 1: Forgetting to Square

❌ Wrong: sin θ + cos θ = 1

✅ Correct: sin²θ + cos²θ = 1

### Mistake 2: Wrong Sign After Square Root

❌ Wrong: If sin²θ = 1/4, then sin θ = 1/2

✅ Correct: sin θ = ±1/2 (check quadrant for sign)

### Mistake 3: Applying Identity Wrong

❌ Wrong: sin²θ - cos²θ = 1

✅ Correct: sin²θ + cos²θ = 1 (it's plus, not minus)

---

## 🌍 Real-World Applications

### 1. Physics - Energy Conservation
In simple harmonic motion:
$$E = \frac{1}{2}kA^2(\sin^2\omega t + \cos^2\omega t) = \frac{1}{2}kA^2$$

Total energy is constant because sin²θ + cos²θ = 1.

### 2. Electrical Engineering
AC power calculations:
$$P = V_{rms}^2/R$$
where V_{rms} involves sin²θ + cos²θ = 1 for time-averaging.

### 3. Computer Graphics
Rotation matrices maintain unit vectors because rotations preserve the identity.

### 4. Navigation
GPS and radar systems use these identities in coordinate transformations.

---

## 📋 Summary Table

### The Three Pythagorean Identities

| # | Identity | Derived From |
|---|----------|--------------|
| 1 | sin²θ + cos²θ = 1 | Unit circle |
| 2 | 1 + tan²θ = sec²θ | #1 ÷ cos²θ |
| 3 | 1 + cot²θ = csc²θ | #1 ÷ sin²θ |

### Useful Rearrangements

| Given | Find | Formula |
|-------|------|---------|
| cos θ | sin²θ | 1 - cos²θ |
| sin θ | cos²θ | 1 - sin²θ |
| tan θ | sec²θ | 1 + tan²θ |
| sec θ | tan²θ | sec²θ - 1 |
| cot θ | csc²θ | 1 + cot²θ |
| csc θ | cot²θ | csc²θ - 1 |

### Factored Forms

| Identity | Factored Form |
|----------|---------------|
| sin²θ - cos²θ | (sin θ + cos θ)(sin θ - cos θ) |
| sec²θ - tan²θ = 1 | (sec θ + tan θ)(sec θ - tan θ) = 1 |
| csc²θ - cot²θ = 1 | (csc θ + cot θ)(csc θ - cot θ) = 1 |

---

## ❓ Quick Revision Questions

1. **Write the three Pythagorean identities from memory.**

2. **If tan θ = 3/4 and θ is in Quadrant I, find sin θ and cos θ using identities.**

3. **Simplify: $\frac{\sin^2\theta}{1 + \cos\theta}$**  
   (Hint: Use 1 - cos²θ = sin²θ and factor)

4. **Prove: $(1 - \cos^2\theta)(1 + \cot^2\theta) = 1$**

5. **If csc θ = 5/4, find cot θ (assuming θ is acute).**

6. **Why is the identity called "Pythagorean"?**

<details>
<summary>Click to see answers</summary>

1. sin²θ + cos²θ = 1  
   1 + tan²θ = sec²θ  
   1 + cot²θ = csc²θ

2. Using 1 + tan²θ = sec²θ:  
   sec²θ = 1 + 9/16 = 25/16  
   sec θ = 5/4, so cos θ = **4/5**  
   sin θ = tan θ · cos θ = (3/4)(4/5) = **3/5**

3. $\frac{\sin^2\theta}{1 + \cos\theta} = \frac{1 - \cos^2\theta}{1 + \cos\theta} = \frac{(1-\cos\theta)(1+\cos\theta)}{1 + \cos\theta}$  
   $= \boxed{1 - \cos\theta}$

4. LHS = sin²θ · csc²θ = sin²θ · (1/sin²θ) = **1** = RHS ✓

5. Using 1 + cot²θ = csc²θ:  
   cot²θ = 25/16 - 1 = 9/16  
   cot θ = **3/4** (positive since θ is acute)

6. It's derived from the Pythagorean theorem (a² + b² = c²) applied to a right triangle with hypotenuse 1, giving sin²θ + cos²θ = 1.

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 3.1 Fundamental Identities](01-fundamental-identities.md) | [Unit 3 Index](README.md) | [3.3 Reciprocal & Quotient Identities →](03-reciprocal-quotient-identities.md) |

---

[← Back to Main Index](../README.md)
