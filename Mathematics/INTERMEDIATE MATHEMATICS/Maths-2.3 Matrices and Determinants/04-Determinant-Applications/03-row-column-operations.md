# Chapter 4.3: Row and Column Operations on Determinants

[← Previous: Determinant of Transpose](02-determinant-of-transpose.md) | [Back to README](../README.md) | [Next: Area of Triangle →](04-area-of-triangle.md)

---

## 📚 Chapter Overview

Row and column operations (elementary operations) are powerful tools for simplifying determinant calculations. Understanding how these operations affect determinants is crucial for efficient computation and for solving systems of equations.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Identify three types of elementary operations
- Determine how each operation affects the determinant
- Apply operations strategically to simplify calculations
- Avoid common mistakes when using operations

---

## 1. Three Types of Elementary Operations

### Overview

```
┌─────────────────────────────────────────────────────────────────┐
│              ELEMENTARY ROW/COLUMN OPERATIONS                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Type 1: INTERCHANGE (Swap)                                    │
│   ─────────────────────────                                     │
│   Swap two rows (or columns)                                    │
│   Notation: R_i ↔ R_j  or  C_i ↔ C_j                           │
│                                                                  │
│   Type 2: SCALAR MULTIPLICATION                                 │
│   ────────────────────────────                                  │
│   Multiply a row (or column) by a non-zero scalar               │
│   Notation: R_i → kR_i  or  C_i → kC_i                         │
│                                                                  │
│   Type 3: ADDITION                                              │
│   ─────────────────                                             │
│   Add a scalar multiple of one row (column) to another          │
│   Notation: R_i → R_i + kR_j  or  C_i → C_i + kC_j             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Type 1: Row/Column Interchange

### Effect on Determinant

> **Swapping two rows (or columns) multiplies the determinant by -1.**

### Notation

$R_i \leftrightarrow R_j$: Swap rows i and j
$C_i \leftrightarrow C_j$: Swap columns i and j

### Example

$$A = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}, \quad |A| = 4 - 6 = -2$$

After $R_1 \leftrightarrow R_2$:

$$A' = \begin{bmatrix} 3 & 4 \\ 1 & 2 \end{bmatrix}, \quad |A'| = 6 - 4 = 2 = -(-2)$$ ✓

### Visual

```
    BEFORE                  AFTER
    
    | a  b |                | c  d |
    | c  d |      →         | a  b |
    
    det = ad - bc           det = cb - da = -(ad - bc)
    
    Sign CHANGES!
```

---

## 3. Type 2: Scalar Multiplication

### Effect on Determinant

> **Multiplying a row (or column) by scalar k multiplies the determinant by k.**

### Notation

$R_i \to kR_i$: Multiply row i by k
$C_j \to kC_j$: Multiply column j by k

### Example

$$\begin{vmatrix} 2 & 4 \\ 1 & 3 \end{vmatrix}$$

Factor 2 from Row 1:

$$= 2\begin{vmatrix} 1 & 2 \\ 1 & 3 \end{vmatrix} = 2(3 - 2) = 2(1) = 2$$

Verify: $2(3) - 4(1) = 6 - 4 = 2$ ✓

### Important Corollary

If you multiply the **entire matrix** by k (all rows):

$$|kA| = k^n|A|$$ for an n×n matrix

### Visual

```
    ORIGINAL                    FACTORED
    
    | ka  kb |                  | a   b |
    | c   d  |     =    k ×     | c   d |
    
    det = k(ad) - k(b)c         k × (ad - bc)
        = k(ad - bc)
```

---

## 4. Type 3: Row/Column Addition

### Effect on Determinant

> **Adding a scalar multiple of one row to another does NOT change the determinant.**

### Notation

$R_i \to R_i + kR_j$: Add k times row j to row i
$C_i \to C_i + kC_j$: Add k times column j to column i

### Example

$$\begin{vmatrix} 2 & 3 \\ 4 & 5 \end{vmatrix} = 10 - 12 = -2$$

Apply $R_2 \to R_2 - 2R_1$:

$$\begin{vmatrix} 2 & 3 \\ 0 & -1 \end{vmatrix} = 2(-1) - 3(0) = -2$$ ✓

Same determinant!

### Visual

```
    BEFORE                      AFTER R₂ → R₂ + kR₁
    
    | a    b  |                | a       b     |
    | c    d  |       →        | c+ka    d+kb  |
    
    det = ad - bc              det = a(d+kb) - b(c+ka)
                                   = ad + kab - bc - kab
                                   = ad - bc  ✓ (unchanged)
```

---

## 5. Summary of Effects

```
┌─────────────────────────────────────────────────────────────────┐
│              EFFECTS ON DETERMINANT                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Operation                    │  Effect on Determinant         │
│   ─────────────────────────────┼────────────────────────────    │
│   R_i ↔ R_j (swap)            │  Multiplied by -1              │
│                                │                                 │
│   R_i → kR_i (scale)          │  Multiplied by k               │
│                                │                                 │
│   R_i → R_i + kR_j (add)      │  UNCHANGED                     │
│                                │                                 │
└─────────────────────────────────────────────────────────────────┘

Same rules apply for column operations!
```

---

## 6. Strategy: Creating Zeros

### The Key Technique

Use Type 3 operations to create zeros, then either:
1. Expand along the row/column with zeros, OR
2. Reduce to triangular form

### Step-by-Step Example

Evaluate: $\begin{vmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 7 & 8 & 10 \end{vmatrix}$

**Step 1**: Use R₁ to eliminate elements in column 1

$R_2 \to R_2 - 4R_1$
$R_3 \to R_3 - 7R_1$

$$\begin{vmatrix} 1 & 2 & 3 \\ 0 & -3 & -6 \\ 0 & -6 & -11 \end{vmatrix}$$

**Step 2**: Factor out -3 from R₂ (Type 2)

$$= (-3)\begin{vmatrix} 1 & 2 & 3 \\ 0 & 1 & 2 \\ 0 & -6 & -11 \end{vmatrix}$$

**Step 3**: $R_3 \to R_3 + 6R_2$

$$= (-3)\begin{vmatrix} 1 & 2 & 3 \\ 0 & 1 & 2 \\ 0 & 0 & 1 \end{vmatrix}$$

**Step 4**: Upper triangular - determinant = product of diagonal

$$= (-3)(1 \times 1 \times 1) = -3$$

---

## 7. Keeping Track of Operations

### Important!

When using row operations, you must track how they affect the determinant:

| If you do... | Then the original det is... |
|--------------|----------------------------|
| $R_i \leftrightarrow R_j$ | $-1 \times$ (new det) |
| $R_i \to kR_i$ | $\frac{1}{k} \times$ (new det) |
| $R_i \to R_i + kR_j$ | Same as new det |

### Example with Tracking

Find: $\begin{vmatrix} 0 & 2 & 3 \\ 1 & 4 & 5 \\ 2 & 6 & 7 \end{vmatrix}$

**Problem**: First element is 0, can't use it as pivot!

**Solution**: Swap R₁ ↔ R₂

$$= -\begin{vmatrix} 1 & 4 & 5 \\ 0 & 2 & 3 \\ 2 & 6 & 7 \end{vmatrix}$$

Now $R_3 \to R_3 - 2R_1$:

$$= -\begin{vmatrix} 1 & 4 & 5 \\ 0 & 2 & 3 \\ 0 & -2 & -3 \end{vmatrix}$$

$R_3 \to R_3 + R_2$:

$$= -\begin{vmatrix} 1 & 4 & 5 \\ 0 & 2 & 3 \\ 0 & 0 & 0 \end{vmatrix}$$

Zero on diagonal → det = 0

$$= -(1 \times 2 \times 0) = 0$$

---

## 8. Useful Factoring Tricks

### Factoring Common Elements

If all elements in a row share a common factor:

$$\begin{vmatrix} 6 & 9 & 12 \\ 2 & 5 & 7 \\ 1 & 3 & 4 \end{vmatrix} = 3\begin{vmatrix} 2 & 3 & 4 \\ 2 & 5 & 7 \\ 1 & 3 & 4 \end{vmatrix}$$

### Factoring from Multiple Rows

$$\begin{vmatrix} 2a & 2b & 2c \\ 3d & 3e & 3f \\ 4g & 4h & 4i \end{vmatrix} = 2 \cdot 3 \cdot 4 \begin{vmatrix} a & b & c \\ d & e & f \\ g & h & i \end{vmatrix} = 24\begin{vmatrix} a & b & c \\ d & e & f \\ g & h & i \end{vmatrix}$$

---

## 9. Column Operations

### Same Rules Apply!

Since $|A^T| = |A|$, all row operation rules work for columns too.

### Example: Column Subtraction

$$\begin{vmatrix} 2 & 4 & 6 \\ 1 & 3 & 5 \\ 0 & 2 & 4 \end{vmatrix}$$

Notice: $C_2 = C_1 + 2$? No. But $C_3 = C_2 + 2$? Check: 6=4+2✓, 5=3+2✓, 4=2+2✓

$C_3 \to C_3 - C_2$:

$$= \begin{vmatrix} 2 & 4 & 2 \\ 1 & 3 & 2 \\ 0 & 2 & 2 \end{vmatrix}$$

Factor 2 from C₃:

$$= 2\begin{vmatrix} 2 & 4 & 1 \\ 1 & 3 & 1 \\ 0 & 2 & 1 \end{vmatrix}$$

$C_2 \to C_2 - C_1$:

$$= 2\begin{vmatrix} 2 & 2 & 1 \\ 1 & 2 & 1 \\ 0 & 2 & 1 \end{vmatrix}$$

Now expand or continue operations...

---

## 10. Common Mistakes

```
┌─────────────────────────────────────────────────────────────────┐
│                     COMMON MISTAKES                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ✗ MISTAKE 1: Forgetting the -1 when swapping rows            │
│     Wrong: After R₁ ↔ R₂, det stays same                       │
│     Right: After R₁ ↔ R₂, det changes sign                     │
│                                                                  │
│   ✗ MISTAKE 2: Not adjusting when factoring                    │
│     Wrong: Factor 3 from a row, det unchanged                  │
│     Right: Factor 3 from a row, det = 3 × (new det)           │
│                                                                  │
│   ✗ MISTAKE 3: R_i → R_i + R_i                                 │
│     This is actually R_i → 2R_i, so det is multiplied by 2!   │
│                                                                  │
│   ✗ MISTAKE 4: Applying operation to both R_i and R_j          │
│     R_i → R_i + R_j and R_j → R_j + R_i simultaneously        │
│     This changes the determinant! Do one at a time.            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 11. Worked Problem

### Problem

Evaluate: $\begin{vmatrix} 1 & 1 & 1 \\ a & b & c \\ a^2 & b^2 & c^2 \end{vmatrix}$ (Vandermonde determinant)

### Solution

$C_2 \to C_2 - C_1$ and $C_3 \to C_3 - C_1$:

$$= \begin{vmatrix} 1 & 0 & 0 \\ a & b-a & c-a \\ a^2 & b^2-a^2 & c^2-a^2 \end{vmatrix}$$

Note: $b^2 - a^2 = (b-a)(b+a)$ and $c^2 - a^2 = (c-a)(c+a)$

Factor $(b-a)$ from C₂ and $(c-a)$ from C₃:

$$= (b-a)(c-a)\begin{vmatrix} 1 & 0 & 0 \\ a & 1 & 1 \\ a^2 & b+a & c+a \end{vmatrix}$$

Expand along Row 1:

$$= (b-a)(c-a) \cdot 1 \cdot \begin{vmatrix} 1 & 1 \\ b+a & c+a \end{vmatrix}$$

$$= (b-a)(c-a)[(c+a) - (b+a)]$$

$$= (b-a)(c-a)(c-b)$$

$$= (a-b)(b-c)(c-a) \cdot (-1)^2 = (a-b)(b-c)(c-a)$$

Or written as: $(b-a)(c-b)(c-a)$

---

## 📊 Summary Table

| Operation | Notation | Effect on det |
|-----------|----------|---------------|
| Swap rows | $R_i \leftrightarrow R_j$ | det → $-$det |
| Swap columns | $C_i \leftrightarrow C_j$ | det → $-$det |
| Multiply row by k | $R_i \to kR_i$ | det → k·det |
| Multiply column by k | $C_j \to kC_j$ | det → k·det |
| Add multiple | $R_i \to R_i + kR_j$ | det → det (unchanged) |
| Add multiple | $C_i \to C_i + kC_j$ | det → det (unchanged) |

---

## ❓ Quick Revision Questions

1. **What happens when you swap two columns?**
   <details>
   <summary>Click for Answer</summary>
   The determinant changes sign (multiplied by -1).
   </details>

2. **If you multiply Row 2 by 5, how does the determinant change?**
   <details>
   <summary>Click for Answer</summary>
   The determinant is multiplied by 5.
   </details>

3. **Does $R_1 \to R_1 + 3R_2$ change the determinant?**
   <details>
   <summary>Click for Answer</summary>
   No, adding a multiple of one row to another leaves the determinant unchanged.
   </details>

4. **How do you handle a zero in the pivot position?**
   <details>
   <summary>Click for Answer</summary>
   Swap rows (or columns) to get a non-zero element in the pivot position. Remember to multiply the result by -1.
   </details>

5. **If you factor 2 from each row of a 3×3 matrix, how does det change?**
   <details>
   <summary>Click for Answer</summary>
   det is multiplied by $2^3 = 8$.
   </details>

6. **What is $\begin{vmatrix} 1 & 2 \\ 3 & 6 \end{vmatrix}$?**
   <details>
   <summary>Click for Answer</summary>
   $C_2 = 2 \cdot C_1$, so columns are proportional. det = 0.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Determinant of Transpose](02-determinant-of-transpose.md) | [Unit 4: Applications](./README.md) | [Area of Triangle →](04-area-of-triangle.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
