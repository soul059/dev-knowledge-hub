# Chapter 7.1: Homogeneous Systems of Linear Equations

[← Previous: Rank and Systems](../06-Rank-of-Matrix/04-rank-and-systems.md) | [Back to README](./README.md) | [Next: Non-Homogeneous Systems →](02-non-homogeneous-systems.md)

---

## 📚 Chapter Overview

Homogeneous systems are special linear systems where all constant terms are zero. They have elegant theoretical properties and are fundamental to understanding the structure of solutions.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Identify homogeneous systems
- Find trivial and non-trivial solutions
- Determine when non-trivial solutions exist
- Describe the solution space

---

## 1. Definition and Basic Properties

### What is a Homogeneous System?

A system of linear equations is **homogeneous** if all constant terms are zero:

$$\begin{cases}
a_{11}x_1 + a_{12}x_2 + \cdots + a_{1n}x_n = 0 \\
a_{21}x_1 + a_{22}x_2 + \cdots + a_{2n}x_n = 0 \\
\vdots \\
a_{m1}x_1 + a_{m2}x_2 + \cdots + a_{mn}x_n = 0
\end{cases}$$

In matrix form: $Ax = 0$

### The Trivial Solution

Every homogeneous system has at least one solution:

$$x_1 = x_2 = \cdots = x_n = 0$$

This is called the **trivial solution**.

```
┌─────────────────────────────────────────────────────────────────┐
│               HOMOGENEOUS SYSTEM Ax = 0                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Key Properties:                                                │
│                                                                  │
│   ✓ Always consistent (never "no solution")                     │
│   ✓ Trivial solution x = 0 always exists                        │
│   ✓ rank(A) = rank([A|0]) always                                │
│                                                                  │
│   The Question: Does a NON-TRIVIAL solution exist?              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Conditions for Non-Trivial Solutions

### Main Theorem

> A homogeneous system $Ax = 0$ has a **non-trivial solution** if and only if $\text{rank}(A) < n$, where n is the number of unknowns.

Equivalently:
- For square matrix: $|A| = 0$
- More unknowns than rank → non-trivial solutions

### Quick Check for Square Systems

For an n×n coefficient matrix A:

| Condition | Solution |
|-----------|----------|
| $\|A\| \neq 0$ | Only trivial solution ($x = 0$) |
| $\|A\| = 0$ | Infinite non-trivial solutions |

---

## 3. Important Corollary

### More Unknowns Than Equations

> If a homogeneous system has more unknowns than equations (n > m), it always has non-trivial solutions.

**Proof**:
- rank(A) ≤ m (number of rows)
- If n > m, then rank(A) < n
- Therefore, non-trivial solutions exist

```
┌─────────────────────────────────────────────────────────────────┐
│        MORE UNKNOWNS THAN EQUATIONS                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   System: 2 equations, 4 unknowns                               │
│                                                                  │
│   x₁ + 2x₂ + x₃ + x₄ = 0                                        │
│   2x₁ + 4x₂ - x₃ + 3x₄ = 0                                      │
│                                                                  │
│   rank(A) ≤ 2 < 4 = n                                           │
│                                                                  │
│   ∴ Non-trivial solutions ALWAYS exist                          │
│   Free variables: at least 4 - 2 = 2                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Solution Space Structure

### The Null Space

The set of all solutions to $Ax = 0$ forms a **vector space** called the **null space** or **kernel** of A.

Properties:
- Contains the zero vector
- Closed under addition: If $x_1$ and $x_2$ are solutions, so is $x_1 + x_2$
- Closed under scalar multiplication: If $x$ is a solution, so is $kx$

### Dimension of Solution Space

$$\dim(\text{null space}) = n - \text{rank}(A) = \text{nullity}(A)$$

---

## 5. Worked Examples

### Example 1: Only Trivial Solution

Solve: $\begin{cases} x + 2y + 3z = 0 \\ 2x + 5y + 3z = 0 \\ x + 8z = 0 \end{cases}$

**Step 1**: Form coefficient matrix

$$A = \begin{bmatrix} 1 & 2 & 3 \\ 2 & 5 & 3 \\ 1 & 0 & 8 \end{bmatrix}$$

**Step 2**: Calculate determinant

$|A| = 1(40 - 0) - 2(16 - 3) + 3(0 - 5)$
$= 40 - 26 - 15 = -1 \neq 0$

**Step 3**: Conclusion

Since $|A| \neq 0$, the only solution is the **trivial solution**:
$$x = 0, \quad y = 0, \quad z = 0$$

---

### Example 2: Non-Trivial Solutions

Solve: $\begin{cases} x + 2y - z = 0 \\ 2x + 4y - 2z = 0 \\ 3x + 6y - 3z = 0 \end{cases}$

**Step 1**: Form augmented matrix and reduce

$$[A|0] = \left[\begin{array}{ccc|c} 1 & 2 & -1 & 0 \\ 2 & 4 & -2 & 0 \\ 3 & 6 & -3 & 0 \end{array}\right]$$

$R_2 \to R_2 - 2R_1$, $R_3 \to R_3 - 3R_1$:

$$\left[\begin{array}{ccc|c} 1 & 2 & -1 & 0 \\ 0 & 0 & 0 & 0 \\ 0 & 0 & 0 & 0 \end{array}\right]$$

**Step 2**: Analyze

- rank(A) = 1
- n = 3 unknowns
- Free variables = 3 - 1 = 2

**Step 3**: Express solution

From $x + 2y - z = 0$: $x = -2y + z$

Let $y = s$ and $z = t$ (free parameters)

$$\boxed{x = -2s + t, \quad y = s, \quad z = t}$$

Or in vector form:
$$\begin{bmatrix} x \\ y \\ z \end{bmatrix} = s\begin{bmatrix} -2 \\ 1 \\ 0 \end{bmatrix} + t\begin{bmatrix} 1 \\ 0 \\ 1 \end{bmatrix}$$

---

### Example 3: Two Equations, Three Unknowns

Solve: $\begin{cases} x + y + z = 0 \\ 2x - y + z = 0 \end{cases}$

**Reduce**:

$$[A|0] = \left[\begin{array}{ccc|c} 1 & 1 & 1 & 0 \\ 2 & -1 & 1 & 0 \end{array}\right]$$

$R_2 \to R_2 - 2R_1$:

$$\left[\begin{array}{ccc|c} 1 & 1 & 1 & 0 \\ 0 & -3 & -1 & 0 \end{array}\right]$$

**Solve**:

From row 2: $-3y - z = 0 \Rightarrow y = -\frac{z}{3}$

From row 1: $x + y + z = 0 \Rightarrow x = -y - z = \frac{z}{3} - z = -\frac{2z}{3}$

Let $z = 3t$ (to avoid fractions):

$$\boxed{x = -2t, \quad y = -t, \quad z = 3t}$$

**Verification**: 
- Equation 1: $-2t + (-t) + 3t = 0$ ✓
- Equation 2: $2(-2t) - (-t) + 3t = -4t + t + 3t = 0$ ✓

---

## 6. Geometric Interpretation

### In Two Variables

$ax + by = 0$ represents a line through the origin.

- Single equation: The line itself is the solution (infinite solutions)
- Two independent equations: Only the origin (trivial solution)
- Two dependent equations: Still the line (infinite solutions)

### In Three Variables

Each equation represents a plane through the origin.

| rank(A) | Solution Set | Geometry |
|---------|--------------|----------|
| 3 | Only (0, 0, 0) | Three planes meet at origin only |
| 2 | Line through origin | Three planes meet along a line |
| 1 | Plane through origin | All three planes coincide |

```
┌─────────────────────────────────────────────────────────────────┐
│         GEOMETRIC INTERPRETATION (3D)                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   rank = 3                rank = 2               rank = 1       │
│   (Only trivial)         (Line solution)      (Plane solution)  │
│                                                                  │
│        ╱│╲                    ╱                    ╱╱╱          │
│       ╱ │ ╲                  ╱│                   ╱╱╱           │
│      ╱──●──╲                ╱ │                  ╱╱╱            │
│      ╲  │  ╱               ╱──●                 ╱╱╱             │
│       ╲ │ ╱                   │╲               ╱╱╱              │
│        ╲│╱                    │ ╲             (same plane)      │
│       (origin)             (line)                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Finding a Basis for Solution Space

### Method

1. Reduce $[A|0]$ to RREF
2. Identify pivot columns and free columns
3. For each free variable, set it to 1 and others to 0
4. Solve for basic variables
5. These vectors form a basis

### Example

Find a basis for the solution space of:
$\begin{cases} x_1 + 2x_2 + x_3 + x_4 = 0 \\ 2x_1 + 4x_2 + 3x_3 + x_4 = 0 \end{cases}$

**Reduce to RREF**:

$$\left[\begin{array}{cccc|c} 1 & 2 & 1 & 1 & 0 \\ 2 & 4 & 3 & 1 & 0 \end{array}\right] \to \left[\begin{array}{cccc|c} 1 & 2 & 0 & 2 & 0 \\ 0 & 0 & 1 & -1 & 0 \end{array}\right]$$

**Identify variables**:
- Pivot columns: 1, 3 → basic variables $x_1, x_3$
- Free columns: 2, 4 → free variables $x_2, x_4$

**Find basis vectors**:

Set $x_2 = 1, x_4 = 0$:
- $x_3 = 0$
- $x_1 = -2$
- Vector: $(-2, 1, 0, 0)$

Set $x_2 = 0, x_4 = 1$:
- $x_3 = 1$
- $x_1 = -2$
- Vector: $(-2, 0, 1, 1)$

**Basis**: $\{(-2, 1, 0, 0), (-2, 0, 1, 1)\}$

---

## 📊 Summary Table

| Condition | Solution Type | Number of Solutions |
|-----------|---------------|---------------------|
| rank(A) = n | Trivial only | 1 (x = 0) |
| rank(A) < n | Non-trivial | Infinite |
| n > m | Non-trivial guaranteed | Infinite |
| $\|A\| \neq 0$ (n×n) | Trivial only | 1 |
| $\|A\| = 0$ (n×n) | Non-trivial | Infinite |

---

## ❓ Quick Revision Questions

1. **What is a homogeneous system?**
   <details>
   <summary>Click for Answer</summary>
   A system Ax = 0 where all constant terms are zero.
   </details>

2. **What is the trivial solution?**
   <details>
   <summary>Click for Answer</summary>
   x₁ = x₂ = ... = xₙ = 0
   </details>

3. **When does a homogeneous system have non-trivial solutions?**
   <details>
   <summary>Click for Answer</summary>
   When rank(A) < n (number of unknowns), or equivalently when |A| = 0 for square systems.
   </details>

4. **3 equations, 5 unknowns, homogeneous. Trivial solution only?**
   <details>
   <summary>Click for Answer</summary>
   No! Since n = 5 > 3 = m, non-trivial solutions always exist.
   </details>

5. **What is the dimension of the solution space?**
   <details>
   <summary>Click for Answer</summary>
   n - rank(A) = nullity(A)
   </details>

6. **If |A| = 5 for a 3×3 system Ax = 0, what is the solution?**
   <details>
   <summary>Click for Answer</summary>
   Only the trivial solution x = y = z = 0.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Rank and Systems](../06-Rank-of-Matrix/04-rank-and-systems.md) | [Unit 7: Systems](./README.md) | [Non-Homogeneous Systems →](02-non-homogeneous-systems.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
