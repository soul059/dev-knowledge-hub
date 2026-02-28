# Chapter 2.2: Scalar Multiplication

[← Previous: Addition and Subtraction](01-addition-and-subtraction.md) | [Back to README](../README.md) | [Next: Matrix Multiplication →](03-matrix-multiplication.md)

---

## 📚 Chapter Overview

Scalar multiplication involves multiplying every element of a matrix by a single number (scalar). This operation is fundamental for scaling matrices and is used extensively in linear algebra and applications.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Perform scalar multiplication on matrices
- Understand properties of scalar multiplication
- Combine scalar multiplication with addition
- Apply scalar multiplication in practical problems

---

## 1. Definition of Scalar Multiplication

### What is a Scalar?

> A **scalar** is a single number (as opposed to a matrix, which is an array of numbers).

Examples of scalars: 2, -5, 0.5, π, √2

### Definition

> **Scalar Multiplication**: To multiply a matrix A by a scalar k, multiply every element of A by k.

### Formal Definition

If $A = [a_{ij}]_{m \times n}$ and $k$ is a scalar, then:

$$kA = [k \cdot a_{ij}]_{m \times n}$$

### Visual Representation

```
                  Scalar            Matrix              Result
                    k         ×        A          =       kA
                    
                              ┌─────────────┐     ┌─────────────┐
                    k    ×    │ a₁₁   a₁₂  │  =  │ ka₁₁  ka₁₂ │
                              │ a₂₁   a₂₂  │     │ ka₂₁  ka₂₂ │
                              └─────────────┘     └─────────────┘
        
        Each element is multiplied by k
```

---

## 2. Basic Examples

### Example 1: Simple Scalar Multiplication

$$A = \begin{bmatrix} 2 & 4 \\ 6 & 8 \end{bmatrix}, \quad k = 3$$

$$3A = \begin{bmatrix} 3 \times 2 & 3 \times 4 \\ 3 \times 6 & 3 \times 8 \end{bmatrix} = \begin{bmatrix} 6 & 12 \\ 18 & 24 \end{bmatrix}$$

### Example 2: Negative Scalar

$$B = \begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \end{bmatrix}, \quad k = -2$$

$$-2B = \begin{bmatrix} -2 & -4 & -6 \\ -8 & -10 & -12 \end{bmatrix}$$

### Example 3: Fractional Scalar

$$C = \begin{bmatrix} 6 & 9 \\ 12 & 15 \end{bmatrix}, \quad k = \frac{1}{3}$$

$$\frac{1}{3}C = \begin{bmatrix} 2 & 3 \\ 4 & 5 \end{bmatrix}$$

### Example 4: Zero Scalar

$$D = \begin{bmatrix} 5 & 7 \\ 3 & 9 \end{bmatrix}, \quad k = 0$$

$$0 \cdot D = \begin{bmatrix} 0 & 0 \\ 0 & 0 \end{bmatrix} = O$$

---

## 3. Step-by-Step Process

```
┌─────────────────────────────────────────────────────────────────┐
│              SCALAR MULTIPLICATION PROCESS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    Given: Scalar k and Matrix A                                 │
│                                                                  │
│    ┌─────────────────────────────────────────────────────┐      │
│    │  Step 1: Identify the scalar k                      │      │
│    └────────────────────────┬────────────────────────────┘      │
│                             │                                    │
│                             ▼                                    │
│    ┌─────────────────────────────────────────────────────┐      │
│    │  Step 2: Take each element aᵢⱼ of the matrix        │      │
│    └────────────────────────┬────────────────────────────┘      │
│                             │                                    │
│                             ▼                                    │
│    ┌─────────────────────────────────────────────────────┐      │
│    │  Step 3: Compute k × aᵢⱼ for each element           │      │
│    └────────────────────────┬────────────────────────────┘      │
│                             │                                    │
│                             ▼                                    │
│    ┌─────────────────────────────────────────────────────┐      │
│    │  Step 4: Place results in corresponding positions   │      │
│    └─────────────────────────────────────────────────────┘      │
│                                                                  │
│    Result: kA has the same order as A                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Properties of Scalar Multiplication

### Property 1: Scalar Distributive (over matrix addition)

> **k(A + B) = kA + kB**

```
Example: k = 2, A = [1 2; 3 4], B = [5 6; 7 8]

Method 1: k(A + B)
A + B = [6 8; 10 12]
2(A + B) = [12 16; 20 24]

Method 2: kA + kB
2A = [2 4; 6 8]
2B = [10 12; 14 16]
2A + 2B = [12 16; 20 24]  ✓
```

### Property 2: Matrix Distributive (over scalar addition)

> **(k + l)A = kA + lA**

```
Example: k = 2, l = 3, A = [1 2; 3 4]

Method 1: (k + l)A
(2 + 3)A = 5A = [5 10; 15 20]

Method 2: kA + lA
2A = [2 4; 6 8]
3A = [3 6; 9 12]
2A + 3A = [5 10; 15 20]  ✓
```

### Property 3: Associative with Scalars

> **(kl)A = k(lA)**

```
Example: k = 2, l = 3, A = [1 2; 3 4]

Method 1: (kl)A
(2 × 3)A = 6A = [6 12; 18 24]

Method 2: k(lA)
3A = [3 6; 9 12]
2(3A) = [6 12; 18 24]  ✓
```

### Property 4: Identity Scalar

> **1 · A = A**

$$1 \cdot \begin{bmatrix} a & b \\ c & d \end{bmatrix} = \begin{bmatrix} a & b \\ c & d \end{bmatrix}$$

### Property 5: Zero Scalar

> **0 · A = O** (zero matrix)

### Property 6: Negative One Scalar

> **(-1)A = -A**

$$(-1) \begin{bmatrix} 2 & 3 \\ 4 & 5 \end{bmatrix} = \begin{bmatrix} -2 & -3 \\ -4 & -5 \end{bmatrix}$$

### Properties Summary

```
┌─────────────────────────────────────────────────────────────────┐
│           PROPERTIES OF SCALAR MULTIPLICATION                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    Property                    Formula                          │
│    ────────────────────────────────────────────────────         │
│    Distributive (matrix)       k(A + B) = kA + kB               │
│    Distributive (scalar)       (k + l)A = kA + lA               │
│    Associative                 (kl)A = k(lA)                    │
│    Identity                    1·A = A                          │
│    Zero                        0·A = O                          │
│    Negative                    (-1)A = -A                       │
│                                                                  │
│    Note: All properties hold for any m×n matrix A              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Combined Operations

### Linear Combinations

A **linear combination** of matrices involves scalar multiplication and addition.

$$\alpha A + \beta B + \gamma C + \cdots$$

### Example 1: Linear Combination

**Problem**: Find 2A - 3B where:

$$A = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}, \quad B = \begin{bmatrix} 0 & 1 \\ 2 & 3 \end{bmatrix}$$

**Solution**:

$$2A = \begin{bmatrix} 2 & 4 \\ 6 & 8 \end{bmatrix}$$

$$3B = \begin{bmatrix} 0 & 3 \\ 6 & 9 \end{bmatrix}$$

$$2A - 3B = \begin{bmatrix} 2-0 & 4-3 \\ 6-6 & 8-9 \end{bmatrix} = \begin{bmatrix} 2 & 1 \\ 0 & -1 \end{bmatrix}$$

### Example 2: Finding Unknown Scalar

**Problem**: Find k if $kA = B$ where:

$$A = \begin{bmatrix} 2 & 4 \\ 6 & 8 \end{bmatrix}, \quad B = \begin{bmatrix} 6 & 12 \\ 18 & 24 \end{bmatrix}$$

**Solution**:
Comparing corresponding elements:
- $2k = 6 \Rightarrow k = 3$
- $4k = 12 \Rightarrow k = 3$
- $6k = 18 \Rightarrow k = 3$
- $8k = 24 \Rightarrow k = 3$

**Answer**: k = 3

### Example 3: Matrix Equation

**Problem**: Find X if $3X + A = B$ where:

$$A = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}, \quad B = \begin{bmatrix} 7 & 11 \\ 15 & 19 \end{bmatrix}$$

**Solution**:

$$3X = B - A = \begin{bmatrix} 7-1 & 11-2 \\ 15-3 & 19-4 \end{bmatrix} = \begin{bmatrix} 6 & 9 \\ 12 & 15 \end{bmatrix}$$

$$X = \frac{1}{3} \begin{bmatrix} 6 & 9 \\ 12 & 15 \end{bmatrix} = \begin{bmatrix} 2 & 3 \\ 4 & 5 \end{bmatrix}$$

---

## 6. Geometric Interpretation

### Scaling Vectors

When a matrix represents a vector (column matrix), scalar multiplication **scales** the vector.

```
        Original Vector          Scaled by 2            Scaled by -1
        
               ▲                       ▲                      │
               │                       │                      │
               │ v                     │ 2v                   ▼
               │                       │                     -v
        ───────┼───────         ───────┼───────         ───────┼───────
               │                       │                      │
               │                       │                      │
               │                       │                      │
               ▼                       ▼                      ▲
        
        v = [2]               2v = [4]              -v = [-2]
            [3]                    [6]                   [-3]
```

### Visual Scaling Example

```
        k = 0.5                   k = 1                    k = 2
        (Shrink)                (Original)               (Stretch)
        
        ┌─────────┐           ┌─────────────┐         ┌─────────────────┐
        │ 0.5  1  │           │  1    2     │         │  2    4         │
        │ 1.5  2  │           │  3    4     │         │  6    8         │
        └─────────┘           └─────────────┘         └─────────────────┘
           ×0.5                    ×1                      ×2
```

---

## 7. Special Cases

### Scalar Multiplication of Identity Matrix

$$kI = \text{Scalar Matrix}$$

$$3I_2 = 3\begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix} = \begin{bmatrix} 3 & 0 \\ 0 & 3 \end{bmatrix}$$

### Scalar Multiplication of Zero Matrix

$$kO = O$$ for any scalar k

### Multiplication by Reciprocal (Division)

$$\frac{1}{k}A = \frac{A}{k}$$ where each element is divided by k

$$\frac{1}{2}\begin{bmatrix} 4 & 6 \\ 8 & 10 \end{bmatrix} = \begin{bmatrix} 2 & 3 \\ 4 & 5 \end{bmatrix}$$

---

## 8. Real-World Applications

### Application 1: Scaling Prices

```
        Original Prices          10% Increase           New Prices
        (in dollars)             (multiply by 1.1)      (in dollars)
        
        ┌──────────────┐                               ┌──────────────┐
        │ 100  50  75  │                               │ 110  55  82.5│
        │  80  60  45  │    ×  1.1  =                  │  88  66  49.5│
        │  90  70  55  │                               │  99  77  60.5│
        └──────────────┘                               └──────────────┘
```

### Application 2: Currency Conversion

```
        Prices in USD           Exchange Rate          Prices in EUR
        
        ┌──────────────┐                               ┌──────────────┐
        │ 100  200  50 │                               │  85  170  42.5│
        │ 150  75  125 │    ×  0.85  =                │127.5 63.75 106.25│
        └──────────────┘                               └──────────────┘
```

### Application 3: Percentage Discounts

```
        Original    25% Discount     After Discount
                    (× 0.75)
        
          40            30
          60    ──→     45
          80            60
         100            75
```

---

## 📊 Summary Table

| Concept | Formula | Example |
|---------|---------|---------|
| Scalar multiplication | $kA = [ka_{ij}]$ | $2\begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix} = \begin{bmatrix} 2 & 4 \\ 6 & 8 \end{bmatrix}$ |
| Negative scalar | $(-k)A = -kA$ | Changes all signs |
| Zero scalar | $0 \cdot A = O$ | Results in zero matrix |
| Distributive (matrix) | $k(A+B) = kA + kB$ | Scalar distributes |
| Distributive (scalar) | $(k+l)A = kA + lA$ | Scalars distribute |
| Associative | $(kl)A = k(lA)$ | Order doesn't matter |

---

## ❓ Quick Revision Questions

1. **Calculate $5 \times \begin{bmatrix} 2 & -1 \\ 0 & 3 \end{bmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   $\begin{bmatrix} 10 & -5 \\ 0 & 15 \end{bmatrix}$
   </details>

2. **If 3A = $\begin{bmatrix} 9 & 12 \\ 6 & 15 \end{bmatrix}$, find A.**
   <details>
   <summary>Click for Answer</summary>
   $A = \frac{1}{3} \times \begin{bmatrix} 9 & 12 \\ 6 & 15 \end{bmatrix} = \begin{bmatrix} 3 & 4 \\ 2 & 5 \end{bmatrix}$
   </details>

3. **Find 2A - B if A = $\begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}$ and B = $\begin{bmatrix} 0 & 1 \\ 1 & 0 \end{bmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   $2A = \begin{bmatrix} 2 & 4 \\ 6 & 8 \end{bmatrix}$, so $2A - B = \begin{bmatrix} 2 & 3 \\ 5 & 8 \end{bmatrix}$
   </details>

4. **What is the result of multiplying any matrix by 0?**
   <details>
   <summary>Click for Answer</summary>
   The zero matrix (O) of the same order.
   </details>

5. **Verify that 2(A + B) = 2A + 2B for A = $\begin{bmatrix} 1 \\ 2 \end{bmatrix}$ and B = $\begin{bmatrix} 3 \\ 4 \end{bmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   $A + B = \begin{bmatrix} 4 \\ 6 \end{bmatrix}$, so $2(A+B) = \begin{bmatrix} 8 \\ 12 \end{bmatrix}$
   $2A = \begin{bmatrix} 2 \\ 4 \end{bmatrix}$, $2B = \begin{bmatrix} 6 \\ 8 \end{bmatrix}$
   $2A + 2B = \begin{bmatrix} 8 \\ 12 \end{bmatrix}$ ✓ Equal!
   </details>

6. **Solve for X: 2X + $\begin{bmatrix} 1 & 1 \\ 1 & 1 \end{bmatrix}$ = $\begin{bmatrix} 5 & 3 \\ 7 & 9 \end{bmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   $2X = \begin{bmatrix} 4 & 2 \\ 6 & 8 \end{bmatrix}$
   $X = \begin{bmatrix} 2 & 1 \\ 3 & 4 \end{bmatrix}$
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Addition and Subtraction](01-addition-and-subtraction.md) | [Unit 2: Operations](./README.md) | [Matrix Multiplication →](03-matrix-multiplication.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
