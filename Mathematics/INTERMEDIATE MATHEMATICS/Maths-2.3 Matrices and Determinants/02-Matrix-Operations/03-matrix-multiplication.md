# Chapter 2.3: Matrix Multiplication

[← Previous: Scalar Multiplication](02-scalar-multiplication.md) | [Back to README](../README.md) | [Next: Properties of Operations →](04-properties-of-operations.md)

---

## 📚 Chapter Overview

Matrix multiplication is one of the most important operations in linear algebra. Unlike addition and scalar multiplication, it involves a specific row-column combination that produces a new matrix.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Determine when matrix multiplication is possible
- Compute the product of two matrices
- Understand the row-column multiplication rule
- Apply matrix multiplication in practical problems

---

## 1. Definition and Compatibility

### When is Multiplication Possible?

> Matrix multiplication AB is defined only when the **number of columns in A equals the number of rows in B**.

### Compatibility Rule

```
        Matrix A          Matrix B          Result C
        (m × n)      ×    (n × p)     =    (m × p)
              ↑            ↑
              └────────────┘
              Must be equal!
              
        "Inner dimensions must match"
        "Outer dimensions give the result size"
```

### Examples of Compatibility

| Matrix A | Matrix B | A×B Possible? | Result Order |
|----------|----------|---------------|--------------|
| 2×3 | 3×4 | ✓ Yes | 2×4 |
| 3×2 | 2×5 | ✓ Yes | 3×5 |
| 2×3 | 2×3 | ✗ No | - |
| 4×4 | 4×4 | ✓ Yes | 4×4 |
| 1×5 | 5×1 | ✓ Yes | 1×1 (scalar) |
| 3×1 | 1×4 | ✓ Yes | 3×4 |

---

## 2. The Row-Column Rule

### How to Multiply

> Each element $c_{ij}$ of the product C = AB is computed by:
> **Multiply corresponding elements of Row i of A and Column j of B, then add the products.**

### Formula

$$c_{ij} = \sum_{k=1}^{n} a_{ik} \cdot b_{kj} = a_{i1}b_{1j} + a_{i2}b_{2j} + \cdots + a_{in}b_{nj}$$

### Visual Explanation

```
                    Matrix B
                    ┌─────────────────┐
                    │ b₁₁ b₁₂ b₁₃ ... │
                    │ b₂₁ b₂₂ b₂₃ ... │
                    │ b₃₁ b₃₂ b₃₃ ... │
                    └─────────────────┘
                         ↓
                    Column j
                    
Matrix A            
┌───────────────┐   ┌─────────────────┐
│ a₁₁ a₁₂ a₁₃  │   │  c₁₁ c₁₂ c₁₃   │
│ a₂₁ a₂₂ a₂₃  │ → │  c₂₁ c₂₂ c₂₃   │  Matrix C = AB
│ a₃₁ a₃₂ a₃₃  │   │  c₃₁ c₃₂ c₃₃   │
└───────────────┘   └─────────────────┘
     ↑
   Row i

c_ij = (Row i of A) · (Column j of B)
     = a_i1·b_1j + a_i2·b_2j + a_i3·b_3j
```

---

## 3. Step-by-Step Examples

### Example 1: 2×2 Matrices

**Problem**: Find AB where:

$$A = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}, \quad B = \begin{bmatrix} 5 & 6 \\ 7 & 8 \end{bmatrix}$$

**Solution**:

```
Check: A is 2×2, B is 2×2
       Inner dimensions: 2 = 2 ✓
       Result: 2×2 matrix
```

**Calculating each element**:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   c₁₁ = Row 1 of A × Column 1 of B                              │
│       = [1  2] · [5]  = 1×5 + 2×7 = 5 + 14 = 19                 │
│                  [7]                                             │
│                                                                  │
│   c₁₂ = Row 1 of A × Column 2 of B                              │
│       = [1  2] · [6]  = 1×6 + 2×8 = 6 + 16 = 22                 │
│                  [8]                                             │
│                                                                  │
│   c₂₁ = Row 2 of A × Column 1 of B                              │
│       = [3  4] · [5]  = 3×5 + 4×7 = 15 + 28 = 43                │
│                  [7]                                             │
│                                                                  │
│   c₂₂ = Row 2 of A × Column 2 of B                              │
│       = [3  4] · [6]  = 3×6 + 4×8 = 18 + 32 = 50                │
│                  [8]                                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Result**:

$$AB = \begin{bmatrix} 19 & 22 \\ 43 & 50 \end{bmatrix}$$

### Example 2: Different Order Matrices

**Problem**: Find AB where:

$$A = \begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \end{bmatrix}_{2×3}, \quad B = \begin{bmatrix} 7 & 8 \\ 9 & 10 \\ 11 & 12 \end{bmatrix}_{3×2}$$

**Solution**:

```
Check: A is 2×3, B is 3×2
       Inner dimensions: 3 = 3 ✓
       Result: 2×2 matrix
```

**Step-by-step calculation**:

```
c₁₁ = 1×7 + 2×9 + 3×11 = 7 + 18 + 33 = 58
c₁₂ = 1×8 + 2×10 + 3×12 = 8 + 20 + 36 = 64
c₂₁ = 4×7 + 5×9 + 6×11 = 28 + 45 + 66 = 139
c₂₂ = 4×8 + 5×10 + 6×12 = 32 + 50 + 72 = 154
```

$$AB = \begin{bmatrix} 58 & 64 \\ 139 & 154 \end{bmatrix}$$

### Example 3: Row × Column (Dot Product)

**Problem**: Find AB where:

$$A = \begin{bmatrix} 1 & 2 & 3 \end{bmatrix}_{1×3}, \quad B = \begin{bmatrix} 4 \\ 5 \\ 6 \end{bmatrix}_{3×1}$$

**Solution**:

```
Check: A is 1×3, B is 3×1
       Inner: 3 = 3 ✓
       Result: 1×1 (a scalar!)
```

$$AB = \begin{bmatrix} 1×4 + 2×5 + 3×6 \end{bmatrix} = \begin{bmatrix} 4 + 10 + 18 \end{bmatrix} = \begin{bmatrix} 32 \end{bmatrix}$$

### Example 4: Column × Row (Outer Product)

**Problem**: Find AB where:

$$A = \begin{bmatrix} 1 \\ 2 \\ 3 \end{bmatrix}_{3×1}, \quad B = \begin{bmatrix} 4 & 5 & 6 \end{bmatrix}_{1×3}$$

**Solution**:

```
Check: A is 3×1, B is 1×3
       Inner: 1 = 1 ✓
       Result: 3×3 matrix!
```

$$AB = \begin{bmatrix} 1×4 & 1×5 & 1×6 \\ 2×4 & 2×5 & 2×6 \\ 3×4 & 3×5 & 3×6 \end{bmatrix} = \begin{bmatrix} 4 & 5 & 6 \\ 8 & 10 & 12 \\ 12 & 15 & 18 \end{bmatrix}$$

---

## 4. Matrix Multiplication Algorithm

```
┌─────────────────────────────────────────────────────────────────┐
│              MATRIX MULTIPLICATION ALGORITHM                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Input: Matrix A (m×n) and Matrix B (n×p)                      │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Step 1: Verify n_A = m_B (columns of A = rows of B)    │   │
│   │          If not, multiplication is undefined             │   │
│   └────────────────────────┬────────────────────────────────┘   │
│                            │                                     │
│                            ▼                                     │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Step 2: Create result matrix C of size m×p             │   │
│   └────────────────────────┬────────────────────────────────┘   │
│                            │                                     │
│                            ▼                                     │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Step 3: For each position (i,j) in C:                  │   │
│   │          c_ij = Σ(k=1 to n) a_ik × b_kj                 │   │
│   │                                                          │   │
│   │          i.e., dot product of Row i of A                │   │
│   │               and Column j of B                          │   │
│   └────────────────────────┬────────────────────────────────┘   │
│                            │                                     │
│                            ▼                                     │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Step 4: Return matrix C                                 │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Multiplication with Identity Matrix

### Property

> **AI = IA = A** (when dimensions are compatible)

### Example

$$A = \begin{bmatrix} 2 & 3 \\ 4 & 5 \end{bmatrix}, \quad I = \begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix}$$

$$AI = \begin{bmatrix} 2×1+3×0 & 2×0+3×1 \\ 4×1+5×0 & 4×0+5×1 \end{bmatrix} = \begin{bmatrix} 2 & 3 \\ 4 & 5 \end{bmatrix} = A$$

```
        Visual: Identity matrix acts like multiplying by 1
        
        A × I = A
        
        [2 3]   [1 0]   [2 3]
        [4 5] × [0 1] = [4 5]
```

---

## 6. Non-Commutativity of Matrix Multiplication

### Important Property

> **AB ≠ BA in general**

Matrix multiplication is **NOT commutative**!

### Example: AB ≠ BA

$$A = \begin{bmatrix} 1 & 2 \\ 0 & 1 \end{bmatrix}, \quad B = \begin{bmatrix} 0 & 1 \\ 1 & 0 \end{bmatrix}$$

$$AB = \begin{bmatrix} 1×0+2×1 & 1×1+2×0 \\ 0×0+1×1 & 0×1+1×0 \end{bmatrix} = \begin{bmatrix} 2 & 1 \\ 1 & 0 \end{bmatrix}$$

$$BA = \begin{bmatrix} 0×1+1×0 & 0×2+1×1 \\ 1×1+0×0 & 1×2+0×1 \end{bmatrix} = \begin{bmatrix} 0 & 1 \\ 1 & 2 \end{bmatrix}$$

**Clearly AB ≠ BA!**

### Cases Where AB May Not Even Equal BA

1. **AB exists but BA doesn't**: A is 2×3, B is 3×4. AB is 2×4, but BA is undefined.
2. **Both exist but different sizes**: A is 2×3, B is 3×2. AB is 2×2, BA is 3×3.
3. **Same size but different values**: As shown above.

---

## 7. Special Multiplication Cases

### Case 1: Zero Matrix Multiplication

$$A \times O = O \times A = O$$

### Case 2: AB = O doesn't mean A = O or B = O

$$A = \begin{bmatrix} 1 & 0 \\ 0 & 0 \end{bmatrix}, \quad B = \begin{bmatrix} 0 & 0 \\ 1 & 0 \end{bmatrix}$$

$$AB = \begin{bmatrix} 1×0+0×1 & 1×0+0×0 \\ 0×0+0×1 & 0×0+0×0 \end{bmatrix} = \begin{bmatrix} 0 & 0 \\ 0 & 0 \end{bmatrix} = O$$

Even though neither A nor B is zero!

### Case 3: Powers of Matrices

For a square matrix A:
- $A^2 = A \times A$
- $A^3 = A \times A \times A$
- $A^n = \underbrace{A \times A \times \cdots \times A}_{n \text{ times}}$
- $A^0 = I$ (identity matrix)

---

## 8. Visualization of Multiplication

### Systematic Calculation Pattern

```
For C = AB where A is 2×3 and B is 3×2:

        B
        ┌─────────┐
        │ b₁₁ b₁₂ │
        │ b₂₁ b₂₂ │ ←── Column 1    Column 2
        │ b₃₁ b₃₂ │
        └─────────┘
           ↓   ↓

A                           C
┌─────────────┐       ┌─────────┐
│ a₁₁ a₁₂ a₁₃ │ ──→   │ c₁₁ c₁₂ │  ← Row 1
│ a₂₁ a₂₂ a₂₃ │ ──→   │ c₂₁ c₂₂ │  ← Row 2
└─────────────┘       └─────────┘
   Row 1                  ↑   ↑
   Row 2               (Row A × Col B)
```

### Calculation Grid

```
┌───────────────────────────────────────────────────────────────┐
│                    MULTIPLICATION GRID                         │
├───────────────────────────────────────────────────────────────┤
│                                                                │
│                          B columns                             │
│                     ┌─────────────────┐                        │
│                     │    j=1    j=2   │                        │
│                     │                 │                        │
│     A rows      i=1 │  [c₁₁]  [c₁₂]  │                        │
│                     │                 │                        │
│                 i=2 │  [c₂₁]  [c₂₂]  │                        │
│                     │                 │                        │
│                     └─────────────────┘                        │
│                                                                │
│     Each c_ij = (Row i of A) · (Column j of B)                │
│                                                                │
└───────────────────────────────────────────────────────────────┘
```

---

## 9. Applications of Matrix Multiplication

### Application 1: Linear Transformations

Rotation by 90° counterclockwise:

$$R = \begin{bmatrix} 0 & -1 \\ 1 & 0 \end{bmatrix}$$

Applying to point (3, 1):

$$R \begin{bmatrix} 3 \\ 1 \end{bmatrix} = \begin{bmatrix} 0×3 + (-1)×1 \\ 1×3 + 0×1 \end{bmatrix} = \begin{bmatrix} -1 \\ 3 \end{bmatrix}$$

### Application 2: Network/Graph Analysis

```
        Adjacency Matrix:           Path Matrix (A²):
        Who can reach whom          2-step connections
        directly?
        
            A  B  C                     A  B  C
        A [ 0  1  0 ]               A [ 1  0  1 ]
        B [ 1  0  1 ]   ×   A   =   B [ 0  2  0 ]
        C [ 0  1  0 ]               C [ 1  0  1 ]
```

### Application 3: Economic Input-Output

```
        Production Matrix:    Resource Vector:    Total Resources:
        (how much of each     (resources per      (needed for all
        input per output)      product)            products)
        
        ┌───────────┐         ┌─────┐            ┌─────┐
        │ 0.2  0.3  │         │ 100 │            │  50 │
        │ 0.4  0.1  │    ×    │  50 │     =      │  45 │
        └───────────┘         └─────┘            └─────┘
```

---

## 📊 Summary Table

| Concept | Rule/Formula | Notes |
|---------|--------------|-------|
| Compatibility | Columns of A = Rows of B | Must match to multiply |
| Result size | (m×n) × (n×p) = (m×p) | Outer dimensions |
| Element formula | $c_{ij} = \sum a_{ik}b_{kj}$ | Row-column dot product |
| Identity property | AI = IA = A | Identity preserves matrix |
| Commutativity | AB ≠ BA (generally) | NOT commutative |
| Zero product | AB = O ⇏ A=O or B=O | Can get zero from non-zero |
| Powers | $A^n = A \times A \times \cdots$ | Only for square matrices |

---

## ❓ Quick Revision Questions

1. **Can we multiply a 3×2 matrix by a 2×4 matrix? What is the result size?**
   <details>
   <summary>Click for Answer</summary>
   Yes! Inner dimensions match (2=2). Result is 3×4.
   </details>

2. **Calculate AB for A = $\begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}$ and B = $\begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   B is the identity matrix, so AB = A = $\begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}$
   </details>

3. **If A is 4×3 and B is 3×2, what are the dimensions of AB and BA?**
   <details>
   <summary>Click for Answer</summary>
   AB: 4×2 (valid)
   BA: 3×3 would need B(3×2) × A(4×3) - inner dimensions 2≠4, so BA is UNDEFINED.
   </details>

4. **Find the product: $\begin{bmatrix} 2 & 3 \end{bmatrix} \times \begin{bmatrix} 4 \\ 5 \end{bmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   $= [2×4 + 3×5] = [8 + 15] = [23]$ (a 1×1 matrix/scalar)
   </details>

5. **Is matrix multiplication commutative? Explain.**
   <details>
   <summary>Click for Answer</summary>
   No! AB ≠ BA in general. Sometimes one exists and the other doesn't, sometimes both exist but have different sizes, and even when both are the same size, the values usually differ.
   </details>

6. **If A² = A, what can you say about matrix A?**
   <details>
   <summary>Click for Answer</summary>
   A is called an idempotent matrix. Examples include the identity matrix I (I² = I) and certain projection matrices.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Scalar Multiplication](02-scalar-multiplication.md) | [Unit 2: Operations](./README.md) | [Properties of Operations →](04-properties-of-operations.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
