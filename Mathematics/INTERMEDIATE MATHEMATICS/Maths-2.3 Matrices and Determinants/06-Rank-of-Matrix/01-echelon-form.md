# Chapter 6.1: Echelon Form of a Matrix

[← Previous: Elementary Matrices](../05-Adjoint-and-Inverse/05-elementary-matrices.md) | [Back to README](../README.md) | [Next: Definition of Rank →](02-definition-of-rank.md)

---

## 📚 Chapter Overview

The echelon form (row echelon form) is a standardized matrix form that reveals key structural information about a matrix. It is fundamental for determining rank, solving linear systems, and understanding linear independence.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Identify matrices in row echelon form
- Identify matrices in reduced row echelon form
- Transform any matrix to echelon form using row operations
- Understand the significance of pivot positions

---

## 1. Row Echelon Form (REF)

### Definition

A matrix is in **Row Echelon Form** if:

1. All rows consisting entirely of zeros are at the bottom
2. In each non-zero row, the leading entry (first non-zero element) is to the right of the leading entry in the row above
3. The leading entry in each row is called a **pivot**

### Visual Pattern

```
┌─────────────────────────────────────────────────────────────────┐
│              ROW ECHELON FORM PATTERN                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────────────────────────────┐                       │
│   │  ■  *  *  *  *  *  *  │   ■ = pivot (first non-zero)       │
│   │  0  ■  *  *  *  *  *  │   * = any value                    │
│   │  0  0  0  ■  *  *  *  │   0 = zero                         │
│   │  0  0  0  0  0  ■  *  │                                    │
│   │  0  0  0  0  0  0  0  │   ← zero row at bottom             │
│   └─────────────────────────────────────┘                       │
│                                                                  │
│   Staircase pattern descending left to right                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Examples

**In Row Echelon Form:**
$$\begin{bmatrix} 2 & 3 & 1 & 5 \\ 0 & 4 & 2 & 1 \\ 0 & 0 & 0 & 3 \end{bmatrix} \quad \begin{bmatrix} 1 & 0 & 2 \\ 0 & 5 & 3 \\ 0 & 0 & 7 \end{bmatrix}$$

**NOT in Row Echelon Form:**
$$\begin{bmatrix} 0 & 3 & 1 \\ 2 & 4 & 2 \\ 0 & 0 & 3 \end{bmatrix} \quad \text{(row 2 has leading entry before row 1)}$$

---

## 2. Reduced Row Echelon Form (RREF)

### Definition

A matrix is in **Reduced Row Echelon Form** if:

1. It is in Row Echelon Form, AND
2. Each pivot (leading entry) is equal to 1
3. Each pivot is the only non-zero entry in its column

### Visual Pattern

```
┌─────────────────────────────────────────────────────────────────┐
│           REDUCED ROW ECHELON FORM PATTERN                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────────────────────────────┐                       │
│   │  1  0  *  0  *  0  *  │   1 = pivot (always 1)             │
│   │  0  1  *  0  *  0  *  │   * = any value (free variable col)│
│   │  0  0  0  1  *  0  *  │   0 = zero                         │
│   │  0  0  0  0  0  1  *  │                                    │
│   │  0  0  0  0  0  0  0  │                                    │
│   └─────────────────────────────────────┘                       │
│                                                                  │
│   Pivot columns have exactly one non-zero entry (the pivot 1)  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Examples

**In RREF:**
$$\begin{bmatrix} 1 & 0 & 3 \\ 0 & 1 & 2 \\ 0 & 0 & 0 \end{bmatrix} \quad \begin{bmatrix} 1 & 2 & 0 & 0 \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

**In REF but NOT RREF:**
$$\begin{bmatrix} 2 & 3 & 1 \\ 0 & 4 & 2 \\ 0 & 0 & 5 \end{bmatrix} \quad \text{(pivots not 1)}$$

$$\begin{bmatrix} 1 & 3 & 1 \\ 0 & 1 & 2 \\ 0 & 0 & 1 \end{bmatrix} \quad \text{(non-zero above pivots)}$$

---

## 3. Gaussian Elimination (to REF)

### Process

```
┌─────────────────────────────────────────────────────────────────┐
│              GAUSSIAN ELIMINATION ALGORITHM                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   1. Start with leftmost non-zero column (pivot column)        │
│                                                                  │
│   2. Select a non-zero entry in this column as pivot           │
│      • If necessary, swap rows to bring pivot to current row   │
│                                                                  │
│   3. Use row operations to create zeros below the pivot        │
│      • Rᵢ → Rᵢ - (aᵢⱼ/pivot) × Rpivot                          │
│                                                                  │
│   4. Move to next row and next pivot column                    │
│                                                                  │
│   5. Repeat until all rows are processed                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Example

Transform to REF: $A = \begin{bmatrix} 1 & 2 & 3 \\ 2 & 5 & 7 \\ 3 & 7 & 9 \end{bmatrix}$

**Step 1**: Pivot = $a_{11} = 1$

**Step 2**: Create zeros below pivot
- $R_2 \to R_2 - 2R_1$: $[2, 5, 7] - 2[1, 2, 3] = [0, 1, 1]$
- $R_3 \to R_3 - 3R_1$: $[3, 7, 9] - 3[1, 2, 3] = [0, 1, 0]$

$$\begin{bmatrix} 1 & 2 & 3 \\ 0 & 1 & 1 \\ 0 & 1 & 0 \end{bmatrix}$$

**Step 3**: Move to row 2, column 2. Pivot = 1.

**Step 4**: Create zeros below
- $R_3 \to R_3 - R_2$: $[0, 1, 0] - [0, 1, 1] = [0, 0, -1]$

$$\text{REF} = \begin{bmatrix} 1 & 2 & 3 \\ 0 & 1 & 1 \\ 0 & 0 & -1 \end{bmatrix}$$

---

## 4. Gauss-Jordan Elimination (to RREF)

### Additional Steps

After reaching REF, continue:

1. Make each pivot equal to 1 (divide row by pivot value)
2. Create zeros **above** each pivot using row operations

### Continuing the Example

From REF: $\begin{bmatrix} 1 & 2 & 3 \\ 0 & 1 & 1 \\ 0 & 0 & -1 \end{bmatrix}$

**Step 5**: Make pivot in R3 equal to 1
- $R_3 \to -R_3$: $[0, 0, 1]$

$$\begin{bmatrix} 1 & 2 & 3 \\ 0 & 1 & 1 \\ 0 & 0 & 1 \end{bmatrix}$$

**Step 6**: Create zeros above pivot in column 3
- $R_2 \to R_2 - R_3$: $[0, 1, 0]$
- $R_1 \to R_1 - 3R_3$: $[1, 2, 0]$

$$\begin{bmatrix} 1 & 2 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 1 \end{bmatrix}$$

**Step 7**: Create zero above pivot in column 2
- $R_1 \to R_1 - 2R_2$: $[1, 0, 0]$

$$\text{RREF} = \begin{bmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 1 \end{bmatrix} = I$$

---

## 5. Pivot Positions

### Definition

A **pivot position** is a location in a matrix that corresponds to a leading 1 in the reduced row echelon form.

A **pivot column** is a column that contains a pivot position.

### Importance

- Number of pivots = Rank of matrix
- Pivot columns correspond to leading variables
- Non-pivot columns correspond to free variables

### Example

$$\text{RREF} = \begin{bmatrix} 1 & 2 & 0 & 0 & 3 \\ 0 & 0 & 1 & 0 & 4 \\ 0 & 0 & 0 & 1 & 5 \\ 0 & 0 & 0 & 0 & 0 \end{bmatrix}$$

- Pivot positions: (1,1), (2,3), (3,4)
- Pivot columns: 1, 3, 4
- Non-pivot columns: 2, 5 (free variables if solving system)

---

## 6. Examples with Different Sizes

### Example 1: 3×4 Matrix

$$A = \begin{bmatrix} 1 & 2 & 1 & 3 \\ 2 & 4 & 3 & 8 \\ 3 & 6 & 4 & 11 \end{bmatrix}$$

$R_2 \to R_2 - 2R_1$, $R_3 \to R_3 - 3R_1$:
$$\begin{bmatrix} 1 & 2 & 1 & 3 \\ 0 & 0 & 1 & 2 \\ 0 & 0 & 1 & 2 \end{bmatrix}$$

$R_3 \to R_3 - R_2$:
$$\text{REF} = \begin{bmatrix} 1 & 2 & 1 & 3 \\ 0 & 0 & 1 & 2 \\ 0 & 0 & 0 & 0 \end{bmatrix}$$

Note: Column 2 has no pivot (not a pivot column).

### Example 2: Matrix with Row Swap Needed

$$A = \begin{bmatrix} 0 & 1 & 2 \\ 1 & 2 & 3 \\ 2 & 3 & 5 \end{bmatrix}$$

$R_1 \leftrightarrow R_2$ (need non-zero pivot):
$$\begin{bmatrix} 1 & 2 & 3 \\ 0 & 1 & 2 \\ 2 & 3 & 5 \end{bmatrix}$$

$R_3 \to R_3 - 2R_1$:
$$\begin{bmatrix} 1 & 2 & 3 \\ 0 & 1 & 2 \\ 0 & -1 & -1 \end{bmatrix}$$

$R_3 \to R_3 + R_2$:
$$\text{REF} = \begin{bmatrix} 1 & 2 & 3 \\ 0 & 1 & 2 \\ 0 & 0 & 1 \end{bmatrix}$$

---

## 7. Uniqueness of RREF

### Important Theorem

> Every matrix has a **unique** reduced row echelon form.

However, the REF (not reduced) is NOT unique—different sequences of row operations can produce different REFs.

### Example

Both are valid REFs of the same matrix:
$$\begin{bmatrix} 2 & 4 \\ 0 & 3 \end{bmatrix} \quad \text{and} \quad \begin{bmatrix} 1 & 2 \\ 0 & 3 \end{bmatrix}$$

But the RREF is unique:
$$\begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix}$$

---

## 8. Applications of Echelon Form

| Application | How Echelon Form Helps |
|------------|------------------------|
| Solving linear systems | Back-substitution from REF or direct reading from RREF |
| Finding rank | Count non-zero rows (pivots) |
| Linear independence | Check if all columns are pivot columns |
| Finding inverse | Augment [A\|I] and reduce to [I\|A⁻¹] |
| Null space | Identify free variables from non-pivot columns |

---

## 📊 Summary Table

| Property | Row Echelon Form (REF) | Reduced REF (RREF) |
|----------|------------------------|-------------------|
| Zero rows | At bottom | At bottom |
| Leading entries | Staircase pattern | Staircase pattern |
| Pivot value | Any non-zero | Must be 1 |
| Above pivots | Any value | Must be 0 |
| Uniqueness | Not unique | Unique |

---

## ❓ Quick Revision Questions

1. **What are the conditions for Row Echelon Form?**
   <details>
   <summary>Click for Answer</summary>
   (1) Zero rows at bottom, (2) Each leading entry is to the right of the one above, (3) All entries below a pivot are zero.
   </details>

2. **What additional conditions make it RREF?**
   <details>
   <summary>Click for Answer</summary>
   Each pivot must be 1, and each pivot must be the only non-zero entry in its column.
   </details>

3. **Is the RREF of a matrix unique?**
   <details>
   <summary>Click for Answer</summary>
   Yes, every matrix has exactly one RREF.
   </details>

4. **What is a pivot column?**
   <details>
   <summary>Click for Answer</summary>
   A column that contains a pivot (leading 1 in RREF, or leading entry in REF).
   </details>

5. **How do you handle a zero in the pivot position?**
   <details>
   <summary>Click for Answer</summary>
   Swap rows to bring a non-zero entry to the pivot position.
   </details>

6. **What does Gaussian elimination produce?**
   <details>
   <summary>Click for Answer</summary>
   Row Echelon Form (REF). Gauss-Jordan elimination produces RREF.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Elementary Matrices](../05-Adjoint-and-Inverse/05-elementary-matrices.md) | [Unit 6: Rank](./README.md) | [Definition of Rank →](02-definition-of-rank.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
