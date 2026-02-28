# Chapter 7.3: General Solutions and Parameters

[← Previous: Non-Homogeneous Systems](02-non-homogeneous-systems.md) | [Back to README](./README.md) | [Course Complete! →](../README.md)

---

## 📚 Chapter Overview

When a system has infinitely many solutions, we express them using parameters (free variables). This chapter covers how to write, verify, and interpret general solutions.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Write solutions in parametric form
- Identify particular and general solutions
- Verify solutions by substitution
- Understand the structure of solution sets

---

## 1. Structure of Solutions

### For Non-Homogeneous Systems

The general solution of $Ax = b$ has the form:

$$x = x_p + x_h$$

where:
- $x_p$ = any **particular solution** of $Ax = b$
- $x_h$ = **general solution** of $Ax = 0$ (homogeneous)

```
┌─────────────────────────────────────────────────────────────────┐
│            STRUCTURE OF GENERAL SOLUTION                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   General solution of Ax = b:                                   │
│                                                                  │
│   ┌─────────────────┐   ┌─────────────────────────────────┐     │
│   │   PARTICULAR    │ + │   HOMOGENEOUS SOLUTION          │     │
│   │   SOLUTION      │   │   (Solution of Ax = 0)          │     │
│   └─────────────────┘   └─────────────────────────────────┘     │
│          ↓                           ↓                          │
│     One specific            Linear combination of              │
│     solution to             basis vectors for                   │
│     Ax = b                  null space                          │
│                                                                  │
│   x = x_p + t₁v₁ + t₂v₂ + ... + tₖvₖ                           │
│                                                                  │
│   where k = n - rank(A) = number of free variables              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Writing Parametric Solutions

### From RREF

1. Reduce to RREF
2. Identify pivot and free variables
3. Express pivot variables in terms of free variables
4. Assign parameters to free variables

### Example 1: Two Free Variables

Solve:
$$\begin{cases} x_1 + 2x_2 - x_3 + x_4 = 2 \\ 2x_1 + 4x_2 + x_3 - x_4 = 1 \end{cases}$$

**Reduce to RREF**:

$$\left[\begin{array}{cccc|c} 1 & 2 & -1 & 1 & 2 \\ 2 & 4 & 1 & -1 & 1 \end{array}\right]$$

$R_2 \to R_2 - 2R_1$:

$$\left[\begin{array}{cccc|c} 1 & 2 & -1 & 1 & 2 \\ 0 & 0 & 3 & -3 & -3 \end{array}\right]$$

$R_2 \to \frac{1}{3}R_2$:

$$\left[\begin{array}{cccc|c} 1 & 2 & -1 & 1 & 2 \\ 0 & 0 & 1 & -1 & -1 \end{array}\right]$$

$R_1 \to R_1 + R_2$:

$$\left[\begin{array}{cccc|c} 1 & 2 & 0 & 0 & 1 \\ 0 & 0 & 1 & -1 & -1 \end{array}\right]$$

**Identify variables**:
- Pivot columns: 1, 3 → basic variables: $x_1, x_3$
- Free columns: 2, 4 → free variables: $x_2, x_4$

**Express solution**:

From row 1: $x_1 + 2x_2 = 1 \Rightarrow x_1 = 1 - 2x_2$
From row 2: $x_3 - x_4 = -1 \Rightarrow x_3 = -1 + x_4$

Let $x_2 = s$ and $x_4 = t$:

$$\boxed{\begin{aligned} x_1 &= 1 - 2s \\ x_2 &= s \\ x_3 &= -1 + t \\ x_4 &= t \end{aligned}}$$

**Vector form**:

$$\begin{bmatrix} x_1 \\ x_2 \\ x_3 \\ x_4 \end{bmatrix} = \begin{bmatrix} 1 \\ 0 \\ -1 \\ 0 \end{bmatrix} + s\begin{bmatrix} -2 \\ 1 \\ 0 \\ 0 \end{bmatrix} + t\begin{bmatrix} 0 \\ 0 \\ 1 \\ 1 \end{bmatrix}$$

Here:
- Particular solution: $(1, 0, -1, 0)$
- Homogeneous solutions: span of $\{(-2, 1, 0, 0), (0, 0, 1, 1)\}$

---

## 3. Particular vs General Solution

### Definitions

| Term | Meaning |
|------|---------|
| Particular solution | Any one specific solution |
| General solution | All solutions (with parameters) |
| Trivial solution | Zero solution (for homogeneous) |
| Null space | Set of all homogeneous solutions |

### Example 2: Finding Particular Solution

Given general solution from Example 1, find particular solutions:

**Setting s = 0, t = 0**: $(1, 0, -1, 0)$
**Setting s = 1, t = 0**: $(-1, 1, -1, 0)$
**Setting s = 0, t = 2**: $(1, 0, 1, 2)$

All are valid particular solutions!

---

## 4. Verification of Solutions

### Method

Substitute the parametric solution back into each original equation.

### Example 3: Complete Verification

Verify that $x_1 = 1 - 2s$, $x_2 = s$, $x_3 = -1 + t$, $x_4 = t$ solves:
- $x_1 + 2x_2 - x_3 + x_4 = 2$
- $2x_1 + 4x_2 + x_3 - x_4 = 1$

**Equation 1**:
$(1 - 2s) + 2(s) - (-1 + t) + (t)$
$= 1 - 2s + 2s + 1 - t + t$
$= 2$ ✓

**Equation 2**:
$2(1 - 2s) + 4(s) + (-1 + t) - (t)$
$= 2 - 4s + 4s - 1 + t - t$
$= 1$ ✓

The solution is verified for all values of s and t.

---

## 5. Geometric Interpretation

### In 3D

| Solution Type | Geometry |
|---------------|----------|
| Unique (0 parameters) | Point |
| 1 parameter | Line |
| 2 parameters | Plane |

```
┌─────────────────────────────────────────────────────────────────┐
│         SOLUTION SET GEOMETRY (3D)                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   0 parameters          1 parameter           2 parameters      │
│   (Unique solution)     (Line)                (Plane)           │
│                                                                  │
│         •                   ╱                   ╱╲              │
│                            ╱                   ╱  ╲             │
│       (point)             ╱                   ╱    ╲            │
│                          ╱                   ╱──────╲           │
│                       (line)                 (plane)            │
│                                                                  │
│   rank = n              rank = n-1            rank = n-2        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Complete Worked Example

### Problem

Solve and express the general solution:
$$\begin{cases} x + y + z = 3 \\ 2x + 3y + z = 5 \\ 3x + 4y + 2z = 8 \end{cases}$$

### Solution

**Step 1: Form augmented matrix**

$$[A|b] = \left[\begin{array}{ccc|c} 1 & 1 & 1 & 3 \\ 2 & 3 & 1 & 5 \\ 3 & 4 & 2 & 8 \end{array}\right]$$

**Step 2: Reduce to RREF**

$R_2 \to R_2 - 2R_1$, $R_3 \to R_3 - 3R_1$:

$$\left[\begin{array}{ccc|c} 1 & 1 & 1 & 3 \\ 0 & 1 & -1 & -1 \\ 0 & 1 & -1 & -1 \end{array}\right]$$

$R_3 \to R_3 - R_2$:

$$\left[\begin{array}{ccc|c} 1 & 1 & 1 & 3 \\ 0 & 1 & -1 & -1 \\ 0 & 0 & 0 & 0 \end{array}\right]$$

$R_1 \to R_1 - R_2$:

$$\left[\begin{array}{ccc|c} 1 & 0 & 2 & 4 \\ 0 & 1 & -1 & -1 \\ 0 & 0 & 0 & 0 \end{array}\right]$$

**Step 3: Identify variables**
- Pivot columns: 1, 2 → basic: x, y
- Free column: 3 → free: z

**Step 4: Express solution**

From row 1: $x + 2z = 4 \Rightarrow x = 4 - 2z$
From row 2: $y - z = -1 \Rightarrow y = -1 + z$

Let $z = t$:

$$\boxed{x = 4 - 2t, \quad y = -1 + t, \quad z = t}$$

**Step 5: Vector form**

$$\begin{bmatrix} x \\ y \\ z \end{bmatrix} = \begin{bmatrix} 4 \\ -1 \\ 0 \end{bmatrix} + t\begin{bmatrix} -2 \\ 1 \\ 1 \end{bmatrix}$$

This is a **line** in 3D space passing through $(4, -1, 0)$ with direction vector $(-2, 1, 1)$.

**Step 6: Verification**

For $t = 0$: $(4, -1, 0)$
- $4 + (-1) + 0 = 3$ ✓
- $8 + (-3) + 0 = 5$ ✓
- $12 + (-4) + 0 = 8$ ✓

For $t = 1$: $(2, 0, 1)$
- $2 + 0 + 1 = 3$ ✓
- $4 + 0 + 1 = 5$ ✓
- $6 + 0 + 2 = 8$ ✓

---

## 7. Special Cases

### Case 1: All Variables Free

If rank(A) = 0 and the system is consistent, all variables are free.

### Case 2: No Free Variables

If rank(A) = n and the system is consistent, there's a unique solution (no parameters needed).

### Case 3: Mixed Systems

Some systems may have constraints that create non-obvious relationships.

---

## 8. Relationship Summary

```
┌─────────────────────────────────────────────────────────────────┐
│          COMPLETE SOLUTION FRAMEWORK                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   System Ax = b                                                  │
│        │                                                         │
│        ↓                                                         │
│   Reduce to RREF                                                 │
│        │                                                         │
│   ┌────┴────────────────────────────────────────┐               │
│   │                                              │               │
│   ↓                                              ↓               │
│   Row [0...0|k]            No such row                          │
│   k ≠ 0?                                                        │
│   │                                              │               │
│   ↓                                              ↓               │
│   NO SOLUTION                             CONSISTENT             │
│                                                  │               │
│                                         ┌────────┴────────┐     │
│                                         ↓                 ↓     │
│                                    Free vars?        No free    │
│                                         │             vars      │
│                                         ↓                 ↓     │
│                                    PARAMETERIZED    UNIQUE      │
│                                    SOLUTION         SOLUTION    │
│                                                                  │
│                                    x = x_p + Σ tᵢvᵢ   x = x_p   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table: Writing Solutions

| Step | Action |
|------|--------|
| 1 | Reduce [A\|b] to RREF |
| 2 | Check consistency (no [0 0...0\|k], k≠0) |
| 3 | Identify pivot columns → basic variables |
| 4 | Identify free columns → free variables |
| 5 | Express basic vars in terms of free vars |
| 6 | Assign parameters (t, s, etc.) to free vars |
| 7 | Write vector form: x = x_p + Σ tᵢvᵢ |
| 8 | Verify by substitution |

---

## 🎓 Course Summary

Congratulations! You have completed the study of **Matrices and Determinants**.

### What You've Learned:

1. **Matrix Basics**: Types, operations, properties
2. **Matrix Operations**: Multiplication, transpose, inverse
3. **Determinants**: Calculation, properties, applications
4. **Adjoint and Inverse**: Methods and properties
5. **Rank**: Definition, calculation, meaning
6. **Systems of Equations**: Complete solution methods

### Key Takeaways:

- Matrices represent linear transformations and systems
- Determinants reveal invertibility and unique solutions
- Rank determines the "dimension" of a matrix's action
- All three concepts unite in solving linear systems

---

## ❓ Quick Revision Questions

1. **What is a particular solution?**
   <details>
   <summary>Click for Answer</summary>
   Any single specific solution to the system (one point in the solution set).
   </details>

2. **How do you find the number of parameters needed?**
   <details>
   <summary>Click for Answer</summary>
   Number of parameters = n - rank(A) = number of free variables.
   </details>

3. **What is the general solution structure for Ax = b?**
   <details>
   <summary>Click for Answer</summary>
   x = x_p + x_h (particular solution + homogeneous solution).
   </details>

4. **How do you verify a parametric solution?**
   <details>
   <summary>Click for Answer</summary>
   Substitute the expressions into each original equation; they should be identically satisfied for all parameter values.
   </details>

5. **What geometry does a 2-parameter solution represent in 3D?**
   <details>
   <summary>Click for Answer</summary>
   A plane.
   </details>

6. **Why is the homogeneous solution part important?**
   <details>
   <summary>Click for Answer</summary>
   It spans all possible solutions; adding any homogeneous solution to a particular solution gives another valid solution.
   </details>

---

## 🔗 Navigation

| Previous | Up | Course Home |
|----------|-------|-------------|
| [← Non-Homogeneous Systems](02-non-homogeneous-systems.md) | [Unit 7: Systems](./README.md) | [📚 Course Home](../README.md) |

---

## 🏆 Course Complete!

You have successfully completed the comprehensive study of Matrices and Determinants. Good luck with your exams!

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
