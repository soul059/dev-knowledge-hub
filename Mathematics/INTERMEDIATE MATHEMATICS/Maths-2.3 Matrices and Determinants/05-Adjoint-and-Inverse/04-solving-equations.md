# Chapter 5.4: Solving Equations Using Inverse

[← Previous: Properties of Inverse](03-properties-of-inverse.md) | [Back to README](../README.md) | [Next: Elementary Matrices →](05-elementary-matrices.md)

---

## 📚 Chapter Overview

Matrix inverses provide a powerful and elegant method for solving systems of linear equations. This chapter explores how to use the inverse matrix method and compares it with other solution techniques.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Express linear systems in matrix form
- Solve systems using matrix inverse
- Handle matrix equations of various forms
- Recognize when the inverse method is appropriate

---

## 1. Matrix Form of Linear Systems

### Converting to Matrix Equation

A system of linear equations:
$$\begin{cases}
a_{11}x_1 + a_{12}x_2 + \cdots + a_{1n}x_n = b_1 \\
a_{21}x_1 + a_{22}x_2 + \cdots + a_{2n}x_n = b_2 \\
\vdots \\
a_{n1}x_1 + a_{n2}x_2 + \cdots + a_{nn}x_n = b_n
\end{cases}$$

Can be written as:
$$AX = B$$

### Components

```
┌─────────────────────────────────────────────────────────────────┐
│                    MATRIX EQUATION AX = B                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌───────────────┐   ┌───────┐     ┌───────┐                  │
│   │ a₁₁ a₁₂ ... │   │  x₁  │     │  b₁  │                     │
│   │ a₂₁ a₂₂ ... │   │  x₂  │     │  b₂  │                     │
│   │  ⋮   ⋮   ⋱  │ × │  ⋮   │  =  │  ⋮   │                     │
│   │ aₙ₁ aₙ₂ ... │   │  xₙ  │     │  bₙ  │                     │
│   └───────────────┘   └───────┘     └───────┘                  │
│          A               X             B                        │
│     (n×n matrix)    (unknowns)    (constants)                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. The Solution Formula

### Main Theorem

> If A is invertible (i.e., $|A| \neq 0$), then the system AX = B has a **unique solution**:
> $$X = A^{-1}B$$

### Derivation

Starting from $AX = B$:
$$A^{-1}(AX) = A^{-1}B$$
$$(A^{-1}A)X = A^{-1}B$$
$$IX = A^{-1}B$$
$$X = A^{-1}B$$

### Important Note

The order matters! We multiply by $A^{-1}$ on the **left**:
$$X = A^{-1}B \quad \text{(correct)}$$
$$X \neq BA^{-1} \quad \text{(incorrect for general matrices)}$$

---

## 3. Step-by-Step Method

### Algorithm

```
┌─────────────────────────────────────────────────────────────────┐
│              SOLVING AX = B USING INVERSE                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Step 1: Write the system in matrix form AX = B                │
│                                                                  │
│   Step 2: Calculate |A|                                         │
│           • If |A| = 0: inverse method fails                    │
│           • If |A| ≠ 0: continue                                │
│                                                                  │
│   Step 3: Find A⁻¹ using:                                       │
│           A⁻¹ = (1/|A|) × adj(A)                                │
│                                                                  │
│   Step 4: Compute X = A⁻¹B                                      │
│                                                                  │
│   Step 5: Verify by substituting back into original equations  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Example: 2×2 System

### Problem

Solve: $\begin{cases} 2x + 3y = 8 \\ 4x + 5y = 14 \end{cases}$

### Solution

**Step 1**: Matrix form
$$\begin{bmatrix} 2 & 3 \\ 4 & 5 \end{bmatrix}\begin{bmatrix} x \\ y \end{bmatrix} = \begin{bmatrix} 8 \\ 14 \end{bmatrix}$$

**Step 2**: Calculate |A|
$$|A| = 2(5) - 3(4) = 10 - 12 = -2 \neq 0$$ ✓

**Step 3**: Find $A^{-1}$
$$A^{-1} = \frac{1}{-2}\begin{bmatrix} 5 & -3 \\ -4 & 2 \end{bmatrix} = \begin{bmatrix} -\frac{5}{2} & \frac{3}{2} \\ 2 & -1 \end{bmatrix}$$

**Step 4**: Compute X = $A^{-1}$B
$$\begin{bmatrix} x \\ y \end{bmatrix} = \begin{bmatrix} -\frac{5}{2} & \frac{3}{2} \\ 2 & -1 \end{bmatrix}\begin{bmatrix} 8 \\ 14 \end{bmatrix}$$

$$= \begin{bmatrix} -\frac{5}{2}(8) + \frac{3}{2}(14) \\ 2(8) + (-1)(14) \end{bmatrix} = \begin{bmatrix} -20 + 21 \\ 16 - 14 \end{bmatrix} = \begin{bmatrix} 1 \\ 2 \end{bmatrix}$$

**Answer**: $x = 1$, $y = 2$

**Verification**: 
- $2(1) + 3(2) = 2 + 6 = 8$ ✓
- $4(1) + 5(2) = 4 + 10 = 14$ ✓

---

## 5. Example: 3×3 System

### Problem

Solve: $\begin{cases} x + 2y + 3z = 14 \\ 2x + 3y + z = 11 \\ 3x + y + 2z = 11 \end{cases}$

### Solution

**Step 1**: Matrix form

$$A = \begin{bmatrix} 1 & 2 & 3 \\ 2 & 3 & 1 \\ 3 & 1 & 2 \end{bmatrix}, \quad B = \begin{bmatrix} 14 \\ 11 \\ 11 \end{bmatrix}$$

**Step 2**: Calculate |A|

Using Sarrus' rule:
- Positive: $1(3)(2) + 2(1)(3) + 3(2)(1) = 6 + 6 + 6 = 18$
- Negative: $3(3)(3) + 1(1)(1) + 2(2)(2) = 27 + 1 + 8 = 36$

$$|A| = 18 - 36 = -18 \neq 0$$ ✓

**Step 3**: Find cofactors

$C_{11} = \begin{vmatrix} 3 & 1 \\ 1 & 2 \end{vmatrix} = 5$, 
$C_{12} = -\begin{vmatrix} 2 & 1 \\ 3 & 2 \end{vmatrix} = -1$, 
$C_{13} = \begin{vmatrix} 2 & 3 \\ 3 & 1 \end{vmatrix} = -7$

$C_{21} = -\begin{vmatrix} 2 & 3 \\ 1 & 2 \end{vmatrix} = -1$, 
$C_{22} = \begin{vmatrix} 1 & 3 \\ 3 & 2 \end{vmatrix} = -7$, 
$C_{23} = -\begin{vmatrix} 1 & 2 \\ 3 & 1 \end{vmatrix} = 5$

$C_{31} = \begin{vmatrix} 2 & 3 \\ 3 & 1 \end{vmatrix} = -7$, 
$C_{32} = -\begin{vmatrix} 1 & 3 \\ 2 & 1 \end{vmatrix} = 5$, 
$C_{33} = \begin{vmatrix} 1 & 2 \\ 2 & 3 \end{vmatrix} = -1$

**Adjoint**:
$$\text{adj}(A) = \begin{bmatrix} 5 & -1 & -7 \\ -1 & -7 & 5 \\ -7 & 5 & -1 \end{bmatrix}$$

**Inverse**:
$$A^{-1} = \frac{1}{-18}\begin{bmatrix} 5 & -1 & -7 \\ -1 & -7 & 5 \\ -7 & 5 & -1 \end{bmatrix}$$

**Step 4**: Compute X

$$X = A^{-1}B = \frac{1}{-18}\begin{bmatrix} 5 & -1 & -7 \\ -1 & -7 & 5 \\ -7 & 5 & -1 \end{bmatrix}\begin{bmatrix} 14 \\ 11 \\ 11 \end{bmatrix}$$

$$= \frac{1}{-18}\begin{bmatrix} 70 - 11 - 77 \\ -14 - 77 + 55 \\ -98 + 55 - 11 \end{bmatrix} = \frac{1}{-18}\begin{bmatrix} -18 \\ -36 \\ -54 \end{bmatrix} = \begin{bmatrix} 1 \\ 2 \\ 3 \end{bmatrix}$$

**Answer**: $x = 1$, $y = 2$, $z = 3$

---

## 6. Other Matrix Equations

### Type 1: XA = B

**Solution**: $X = BA^{-1}$

(Multiply by $A^{-1}$ on the **right**)

### Type 2: AXB = C

**Solution**: $X = A^{-1}CB^{-1}$

(Multiply by $A^{-1}$ on left, $B^{-1}$ on right)

### Type 3: AX = B, XA = C

Finding X such that both hold simultaneously (may not have a solution)

---

## 7. Example: XA = B Type

### Problem

Find X if $XA = B$ where:
$$A = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}, \quad B = \begin{bmatrix} 5 & 6 \\ 7 & 8 \end{bmatrix}$$

### Solution

$$X = BA^{-1}$$

First, find $A^{-1}$:
$$|A| = 4 - 6 = -2$$
$$A^{-1} = \frac{1}{-2}\begin{bmatrix} 4 & -2 \\ -3 & 1 \end{bmatrix} = \begin{bmatrix} -2 & 1 \\ \frac{3}{2} & -\frac{1}{2} \end{bmatrix}$$

Then:
$$X = \begin{bmatrix} 5 & 6 \\ 7 & 8 \end{bmatrix}\begin{bmatrix} -2 & 1 \\ \frac{3}{2} & -\frac{1}{2} \end{bmatrix}$$

$$= \begin{bmatrix} -10+9 & 5-3 \\ -14+12 & 7-4 \end{bmatrix} = \begin{bmatrix} -1 & 2 \\ -2 & 3 \end{bmatrix}$$

---

## 8. Multiple Right-Hand Sides

### Advantage of Inverse Method

If you need to solve $AX = B_1$, $AX = B_2$, ..., $AX = B_k$ with the same A:

1. Compute $A^{-1}$ once
2. For each $B_i$: $X_i = A^{-1}B_i$

This is more efficient than solving each system separately!

---

## 9. When Inverse Method Fails

### Case: |A| = 0 (Singular Matrix)

When $|A| = 0$, the system either has:
- **No solution** (inconsistent)
- **Infinitely many solutions** (dependent)

### Example

$$\begin{cases} x + 2y = 5 \\ 2x + 4y = 10 \end{cases}$$

$$A = \begin{bmatrix} 1 & 2 \\ 2 & 4 \end{bmatrix}, \quad |A| = 4 - 4 = 0$$

The equations are multiples of each other → infinitely many solutions.

---

## 10. Comparison of Methods

```
┌─────────────────────────────────────────────────────────────────┐
│                METHOD COMPARISON                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   INVERSE METHOD (X = A⁻¹B):                                    │
│   ✓ Direct closed-form solution                                 │
│   ✓ Good for multiple B vectors                                 │
│   ✓ Theoretical importance                                      │
│   ✗ Expensive for large matrices                                │
│   ✗ Fails if |A| = 0                                           │
│                                                                  │
│   CRAMER'S RULE:                                                │
│   ✓ Closed-form solution                                        │
│   ✓ Good for symbolic solutions                                 │
│   ✗ Very slow for n > 3                                        │
│                                                                  │
│   GAUSSIAN ELIMINATION:                                         │
│   ✓ Most efficient computationally                             │
│   ✓ Handles singular cases gracefully                          │
│   ✓ Works for rectangular systems                              │
│   ✗ No closed-form expression                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Equation Type | Solution | Condition |
|---------------|----------|-----------|
| AX = B | $X = A^{-1}B$ | $\|A\| \neq 0$ |
| XA = B | $X = BA^{-1}$ | $\|A\| \neq 0$ |
| AXB = C | $X = A^{-1}CB^{-1}$ | $\|A\|, \|B\| \neq 0$ |
| AX = O (homogeneous) | X = O only | if $\|A\| \neq 0$ |

---

## ❓ Quick Revision Questions

1. **What is the solution to AX = B?**
   <details>
   <summary>Click for Answer</summary>
   $X = A^{-1}B$ (when A is invertible)
   </details>

2. **What is the solution to XA = B?**
   <details>
   <summary>Click for Answer</summary>
   $X = BA^{-1}$ (multiply by inverse on the right)
   </details>

3. **When does the inverse method fail?**
   <details>
   <summary>Click for Answer</summary>
   When $|A| = 0$ (A is singular)
   </details>

4. **Solve: $\begin{bmatrix} 1 & 1 \\ 1 & 2 \end{bmatrix}\begin{bmatrix} x \\ y \end{bmatrix} = \begin{bmatrix} 3 \\ 4 \end{bmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   $|A| = 1$, $A^{-1} = \begin{bmatrix} 2 & -1 \\ -1 & 1 \end{bmatrix}$, $X = \begin{bmatrix} 2 \\ 1 \end{bmatrix}$, so $x=2$, $y=1$
   </details>

5. **Why use the inverse method for multiple B vectors?**
   <details>
   <summary>Click for Answer</summary>
   You only compute $A^{-1}$ once, then multiply by each B. This is efficient when solving many systems with the same coefficient matrix.
   </details>

6. **What is the solution to AX = O when |A| ≠ 0?**
   <details>
   <summary>Click for Answer</summary>
   Only the trivial solution: X = O (zero vector)
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Properties of Inverse](03-properties-of-inverse.md) | [Unit 5: Adjoint & Inverse](./README.md) | [Elementary Matrices →](05-elementary-matrices.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
