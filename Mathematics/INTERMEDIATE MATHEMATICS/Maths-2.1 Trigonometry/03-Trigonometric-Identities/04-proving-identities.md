# Chapter 3.4: Proving Trigonometric Identities

## Overview

Proving trigonometric identities is a fundamental skill that tests your understanding of all trigonometric relationships. Unlike solving equations, proving identities requires showing that two expressions are equivalent for all valid values of the variable. This chapter covers strategies, techniques, and common patterns.

---

## 📐 What is a Trigonometric Identity?

### Definition

A **trigonometric identity** is an equation involving trigonometric functions that is true for **all values** of the variable(s) for which both sides are defined.

### Identity vs. Equation

```
    ┌────────────────────────────────────────────────────────────┐
    │           IDENTITY vs EQUATION                             │
    ├────────────────────────────────────────────────────────────┤
    │                                                            │
    │  IDENTITY: True for ALL valid values                       │
    │  Example: sin²θ + cos²θ = 1                                │
    │  (True for θ = 0°, 30°, 45°, 90°, ... any angle)          │
    │                                                            │
    │  EQUATION: True for SOME specific values                   │
    │  Example: sin θ = 1/2                                      │
    │  (Only true when θ = 30°, 150°, etc.)                     │
    │                                                            │
    └────────────────────────────────────────────────────────────┘
```

---

## 🔑 Fundamental Approach

### The Golden Rule

**NEVER treat an identity like an equation!**

❌ **Wrong approach:** Cross-multiply, add to both sides, etc.

✅ **Correct approach:** Transform one side to match the other, OR transform both sides to a common expression.

### Three Valid Methods

```
    Method 1: Transform LHS → RHS
    ─────────────────────────────
    Start with Left-Hand Side
    Apply identities step by step
    Arrive at Right-Hand Side
    
    Method 2: Transform RHS → LHS
    ─────────────────────────────
    Start with Right-Hand Side
    Apply identities step by step
    Arrive at Left-Hand Side
    
    Method 3: Transform Both Sides → Common Expression
    ──────────────────────────────────────────────────
    Work on LHS and RHS separately
    Show both equal the same expression
```

---

## 📊 Strategy Toolkit

### Strategy 1: Convert to Sine and Cosine

The most universally applicable strategy.

```
    Convert:
    • tan θ → sin θ/cos θ
    • cot θ → cos θ/sin θ
    • sec θ → 1/cos θ
    • csc θ → 1/sin θ
```

### Strategy 2: Factor Expressions

Look for:
- Common factors
- Difference of squares: a² - b² = (a+b)(a-b)
- Perfect squares: a² + 2ab + b² = (a+b)²

### Strategy 3: Use Pythagorean Identities

| If You See | Consider Using |
|------------|----------------|
| sin²θ + cos²θ | = 1 |
| 1 - sin²θ | = cos²θ |
| 1 - cos²θ | = sin²θ |
| sec²θ - tan²θ | = 1 |
| csc²θ - cot²θ | = 1 |
| sec²θ - 1 | = tan²θ |
| csc²θ - 1 | = cot²θ |

### Strategy 4: Combine Fractions

Find common denominators to combine terms.

### Strategy 5: Work with the More Complex Side

Start with the side that has:
- More terms
- More complex operations
- Functions you can simplify

### Strategy 6: Multiply by Conjugates

Useful when you have expressions like 1 ± sin θ or 1 ± cos θ.

---

## 🧮 Worked Examples

### Example 1: Basic Conversion (Strategy 1)

**Prove:** $\tan\theta \cdot \cos\theta = \sin\theta$

**Proof:**
$$LHS = \tan\theta \cdot \cos\theta$$
$$= \frac{\sin\theta}{\cos\theta} \cdot \cos\theta$$
$$= \sin\theta = RHS \quad \checkmark$$

### Example 2: Using Pythagorean Identity (Strategy 3)

**Prove:** $\sec^2\theta - \tan^2\theta = 1$

**Proof:**
$$LHS = \sec^2\theta - \tan^2\theta$$
$$= \frac{1}{\cos^2\theta} - \frac{\sin^2\theta}{\cos^2\theta}$$
$$= \frac{1 - \sin^2\theta}{\cos^2\theta}$$
$$= \frac{\cos^2\theta}{\cos^2\theta}$$
$$= 1 = RHS \quad \checkmark$$

### Example 3: Factoring (Strategy 2)

**Prove:** $\sin^4\theta - \cos^4\theta = \sin^2\theta - \cos^2\theta$

**Proof:**
$$LHS = \sin^4\theta - \cos^4\theta$$
$$= (\sin^2\theta)^2 - (\cos^2\theta)^2$$
$$= (\sin^2\theta + \cos^2\theta)(\sin^2\theta - \cos^2\theta)$$
$$= (1)(\sin^2\theta - \cos^2\theta)$$
$$= \sin^2\theta - \cos^2\theta = RHS \quad \checkmark$$

### Example 4: Combining Fractions (Strategy 4)

**Prove:** $\frac{1}{1-\sin\theta} + \frac{1}{1+\sin\theta} = 2\sec^2\theta$

**Proof:**
$$LHS = \frac{1}{1-\sin\theta} + \frac{1}{1+\sin\theta}$$
$$= \frac{(1+\sin\theta) + (1-\sin\theta)}{(1-\sin\theta)(1+\sin\theta)}$$
$$= \frac{2}{1-\sin^2\theta}$$
$$= \frac{2}{\cos^2\theta}$$
$$= 2\sec^2\theta = RHS \quad \checkmark$$

### Example 5: Using Conjugates (Strategy 6)

**Prove:** $\frac{1-\sin\theta}{\cos\theta} = \frac{\cos\theta}{1+\sin\theta}$

**Proof (Method: Cross-multiplication pattern via conjugate):**

Multiply LHS numerator and denominator by (1 + sin θ):
$$LHS = \frac{1-\sin\theta}{\cos\theta} \cdot \frac{1+\sin\theta}{1+\sin\theta}$$
$$= \frac{(1-\sin\theta)(1+\sin\theta)}{\cos\theta(1+\sin\theta)}$$
$$= \frac{1-\sin^2\theta}{\cos\theta(1+\sin\theta)}$$
$$= \frac{\cos^2\theta}{\cos\theta(1+\sin\theta)}$$
$$= \frac{\cos\theta}{1+\sin\theta} = RHS \quad \checkmark$$

### Example 6: Complex Identity

**Prove:** $\frac{\sin\theta}{1+\cos\theta} + \frac{1+\cos\theta}{\sin\theta} = 2\csc\theta$

**Proof:**
$$LHS = \frac{\sin\theta}{1+\cos\theta} + \frac{1+\cos\theta}{\sin\theta}$$

Combine with common denominator:
$$= \frac{\sin^2\theta + (1+\cos\theta)^2}{\sin\theta(1+\cos\theta)}$$

Expand numerator:
$$= \frac{\sin^2\theta + 1 + 2\cos\theta + \cos^2\theta}{\sin\theta(1+\cos\theta)}$$

Use sin²θ + cos²θ = 1:
$$= \frac{1 + 1 + 2\cos\theta}{\sin\theta(1+\cos\theta)}$$
$$= \frac{2 + 2\cos\theta}{\sin\theta(1+\cos\theta)}$$
$$= \frac{2(1 + \cos\theta)}{\sin\theta(1+\cos\theta)}$$
$$= \frac{2}{\sin\theta}$$
$$= 2\csc\theta = RHS \quad \checkmark$$

---

## 📋 Common Identity Patterns

### Pattern 1: Sum and Product Forms

$$\tan\theta + \cot\theta = \frac{1}{\sin\theta\cos\theta} = \sec\theta\csc\theta$$

$$\tan\theta - \cot\theta = \frac{\sin^2\theta - \cos^2\theta}{\sin\theta\cos\theta}$$

### Pattern 2: Difference of Squares

$$(1+\sin\theta)(1-\sin\theta) = 1 - \sin^2\theta = \cos^2\theta$$

$$(sec\theta + \tan\theta)(\sec\theta - \tan\theta) = \sec^2\theta - \tan^2\theta = 1$$

### Pattern 3: Perfect Squares

$$(\sin\theta + \cos\theta)^2 = 1 + 2\sin\theta\cos\theta$$

$$(\sin\theta - \cos\theta)^2 = 1 - 2\sin\theta\cos\theta$$

---

## 🔍 Tips for Success

### Do's

```
    ✓ Work on one side at a time
    ✓ Show each step clearly
    ✓ Look for opportunities to use known identities
    ✓ Try converting to sin and cos if stuck
    ✓ Factor when possible
    ✓ Look for conjugate opportunities
    ✓ Check your work by testing with specific values
```

### Don'ts

```
    ✗ Don't cross-multiply
    ✗ Don't add/subtract the same thing to both sides
    ✗ Don't work on both sides simultaneously in one equation
    ✗ Don't give up too quickly—try different strategies
```

---

## 📊 Decision Flowchart

```
    START: Given identity to prove
           │
           ▼
    ┌──────────────────────┐
    │ Which side is more   │
    │ complex?             │
    └──────────────────────┘
           │
           ▼
    Start with that side
           │
           ▼
    ┌──────────────────────┐
    │ Are there multiple   │──Yes──→ Convert all to sin/cos
    │ trig functions?      │
    └──────────────────────┘
           │ No
           ▼
    ┌──────────────────────┐
    │ Is there a sum of    │──Yes──→ Find common denominator
    │ fractions?           │
    └──────────────────────┘
           │ No
           ▼
    ┌──────────────────────┐
    │ Are there squared    │──Yes──→ Try Pythagorean identities
    │ terms?               │
    └──────────────────────┘
           │ No
           ▼
    ┌──────────────────────┐
    │ Is there 1±sin or    │──Yes──→ Try conjugate multiplication
    │ 1±cos?               │
    └──────────────────────┘
           │ No
           ▼
    Try factoring or other algebraic manipulation
           │
           ▼
        END: Identity proven
```

---

## 🧪 Practice Problems with Solutions

### Problem 1
**Prove:** $\csc\theta - \sin\theta = \cot\theta\cos\theta$

<details>
<summary>Solution</summary>

$$LHS = \csc\theta - \sin\theta$$
$$= \frac{1}{\sin\theta} - \sin\theta$$
$$= \frac{1 - \sin^2\theta}{\sin\theta}$$
$$= \frac{\cos^2\theta}{\sin\theta}$$
$$= \frac{\cos\theta}{\sin\theta} \cdot \cos\theta$$
$$= \cot\theta \cdot \cos\theta = RHS \quad \checkmark$$

</details>

### Problem 2
**Prove:** $\frac{\tan^2\theta}{1+\tan^2\theta} = \sin^2\theta$

<details>
<summary>Solution</summary>

$$LHS = \frac{\tan^2\theta}{1+\tan^2\theta}$$
$$= \frac{\tan^2\theta}{\sec^2\theta}$$
$$= \tan^2\theta \cdot \cos^2\theta$$
$$= \frac{\sin^2\theta}{\cos^2\theta} \cdot \cos^2\theta$$
$$= \sin^2\theta = RHS \quad \checkmark$$

</details>

### Problem 3
**Prove:** $\frac{1+\tan^2\theta}{1+\cot^2\theta} = \tan^2\theta$

<details>
<summary>Solution</summary>

$$LHS = \frac{1+\tan^2\theta}{1+\cot^2\theta}$$
$$= \frac{\sec^2\theta}{\csc^2\theta}$$
$$= \frac{1/\cos^2\theta}{1/\sin^2\theta}$$
$$= \frac{\sin^2\theta}{\cos^2\theta}$$
$$= \tan^2\theta = RHS \quad \checkmark$$

</details>

---

## 🌍 Real-World Applications

### 1. Simplifying Complex Expressions
Engineers use identity proofs to simplify circuit analysis equations.

### 2. Computer Algebra Systems
Software uses identity rules to simplify user expressions.

### 3. Physics Derivations
Proving identities is essential for deriving physics formulas.

### 4. Signal Processing
Fourier analysis relies heavily on trigonometric identity manipulation.

---

## 📋 Summary Table

### Strategies Ranked by Usefulness

| Rank | Strategy | When to Use |
|------|----------|-------------|
| 1 | Convert to sin/cos | Multiple different functions |
| 2 | Pythagorean identities | Squared terms |
| 3 | Common denominator | Sum of fractions |
| 4 | Factoring | Difference of squares, common factors |
| 5 | Conjugate multiplication | Expressions with 1 ± sin θ or 1 ± cos θ |
| 6 | Work both sides | When one side alone is difficult |

### Key Identities for Proofs

| Identity | Common Use |
|----------|------------|
| sin²θ + cos²θ = 1 | Simplifying squared terms |
| 1 - sin²θ = cos²θ | Converting sin² to cos² |
| 1 - cos²θ = sin²θ | Converting cos² to sin² |
| 1 + tan²θ = sec²θ | Simplifying with tan and sec |
| 1 + cot²θ = csc²θ | Simplifying with cot and csc |
| (a+b)(a-b) = a²-b² | Factoring |

---

## ❓ Quick Revision Questions

1. **Why can't you cross-multiply when proving an identity?**

2. **Prove: $\cos\theta \cdot \tan\theta = \sin\theta$**

3. **Prove: $(1 - \cos^2\theta)(1 + \cot^2\theta) = 1$**

4. **What strategy would you use for: $\frac{\sin\theta}{1-\cos\theta} = \csc\theta + \cot\theta$?**

5. **Prove: $\sec\theta + \tan\theta = \frac{1}{\sec\theta - \tan\theta}$**

6. **Which side should you start with when proving $\frac{1+\sin\theta}{\cos\theta} = \frac{\cos\theta}{1-\sin\theta}$?**

<details>
<summary>Click to see answers</summary>

1. Because cross-multiplying assumes the identity is true (what we're trying to prove). It's circular reasoning. We must transform one side independently.

2. LHS = cos θ · (sin θ/cos θ) = sin θ = RHS ✓

3. LHS = sin²θ · csc²θ = sin²θ · (1/sin²θ) = 1 = RHS ✓

4. **Conjugate multiplication** - multiply LHS by (1+cos θ)/(1+cos θ) to get a difference of squares in the denominator.

5. Multiply RHS by (sec θ + tan θ)/(sec θ + tan θ):
   RHS = (sec θ + tan θ)/[(sec θ - tan θ)(sec θ + tan θ)]
   = (sec θ + tan θ)/(sec²θ - tan²θ)
   = (sec θ + tan θ)/1 = LHS ✓

6. **Either side** works with conjugate multiplication. Starting with LHS: multiply by (1-sin θ)/(1-sin θ) gives cos²θ/(cos θ(1-sin θ)) = cos θ/(1-sin θ).

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 3.3 Reciprocal & Quotient Identities](03-reciprocal-quotient-identities.md) | [Unit 3 Index](README.md) | [Unit 4: Compound Angles →](../04-Compound-Angles/README.md) |

---

[← Back to Main Index](../README.md)
