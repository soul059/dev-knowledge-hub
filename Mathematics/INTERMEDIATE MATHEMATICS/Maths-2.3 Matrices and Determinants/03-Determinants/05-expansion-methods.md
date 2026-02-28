# Chapter 3.5: Expansion Methods

[← Previous: Minor and Cofactor](04-minor-and-cofactor.md) | [Back to README](../README.md) | [Next: Product of Determinants →](../04-Determinant-Applications/01-product-of-determinants.md)

---

## 📚 Chapter Overview

This chapter covers various methods for evaluating determinants. We'll explore cofactor expansion, Sarrus' rule (for 3×3), and techniques using row/column operations to simplify calculations.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Apply cofactor expansion along any row or column
- Choose the optimal row/column for expansion
- Combine row operations with expansion for efficiency
- Evaluate higher-order determinants systematically

---

## 1. Cofactor Expansion Method

### The Fundamental Formula

#### Row Expansion (along row i)

$$|A| = \sum_{j=1}^{n} a_{ij} \cdot C_{ij}$$

#### Column Expansion (along column j)

$$|A| = \sum_{i=1}^{n} a_{ij} \cdot C_{ij}$$

### Key Insight

```
┌─────────────────────────────────────────────────────────────────┐
│                    EXPANSION STRATEGY                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   The result is the SAME regardless of which row or column      │
│   you choose for expansion!                                      │
│                                                                  │
│   Strategy: Choose the row or column with the most zeros        │
│   to minimize calculations.                                      │
│                                                                  │
│   Each zero element contributes 0 × C_ij = 0 to the sum         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Expansion Along First Row

### Standard 3×3 Expansion

For:
$$A = \begin{bmatrix} a_{11} & a_{12} & a_{13} \\ a_{21} & a_{22} & a_{23} \\ a_{31} & a_{32} & a_{33} \end{bmatrix}$$

Expanding along Row 1:

$$|A| = a_{11}C_{11} + a_{12}C_{12} + a_{13}C_{13}$$

$$= a_{11}(-1)^{1+1}\begin{vmatrix} a_{22} & a_{23} \\ a_{32} & a_{33} \end{vmatrix} + a_{12}(-1)^{1+2}\begin{vmatrix} a_{21} & a_{23} \\ a_{31} & a_{33} \end{vmatrix} + a_{13}(-1)^{1+3}\begin{vmatrix} a_{21} & a_{22} \\ a_{31} & a_{32} \end{vmatrix}$$

$$= a_{11}(a_{22}a_{33} - a_{23}a_{32}) - a_{12}(a_{21}a_{33} - a_{23}a_{31}) + a_{13}(a_{21}a_{32} - a_{22}a_{31})$$

### Visual Pattern for Row 1 Expansion

```
    | a  b  c |
    | d  e  f |    =  a|e f| - b|d f| + c|d e|
    | g  h  i |        |h i|    |g i|    |g h|
    
                   =  a(ei-fh) - b(di-fg) + c(dh-eg)
```

---

## 3. Worked Example: Row 1 Expansion

### Problem

Evaluate: $\begin{vmatrix} 2 & 3 & 1 \\ 4 & -1 & 5 \\ 6 & 2 & 7 \end{vmatrix}$

### Solution

Step 1: Expand along Row 1

$$|A| = 2 \cdot C_{11} + 3 \cdot C_{12} + 1 \cdot C_{13}$$

Step 2: Calculate each cofactor

$$C_{11} = (+1)\begin{vmatrix} -1 & 5 \\ 2 & 7 \end{vmatrix} = -7 - 10 = -17$$

$$C_{12} = (-1)\begin{vmatrix} 4 & 5 \\ 6 & 7 \end{vmatrix} = -(28 - 30) = 2$$

$$C_{13} = (+1)\begin{vmatrix} 4 & -1 \\ 6 & 2 \end{vmatrix} = 8 - (-6) = 14$$

Step 3: Combine

$$|A| = 2(-17) + 3(2) + 1(14)$$
$$= -34 + 6 + 14$$
$$= -14$$

---

## 4. Expansion Along Other Rows/Columns

### Example: Column 1 Expansion

Same matrix: $\begin{vmatrix} 2 & 3 & 1 \\ 4 & -1 & 5 \\ 6 & 2 & 7 \end{vmatrix}$

Expanding along Column 1:

$$|A| = a_{11}C_{11} + a_{21}C_{21} + a_{31}C_{31}$$

$$= 2(+1)\begin{vmatrix} -1 & 5 \\ 2 & 7 \end{vmatrix} + 4(-1)\begin{vmatrix} 3 & 1 \\ 2 & 7 \end{vmatrix} + 6(+1)\begin{vmatrix} 3 & 1 \\ -1 & 5 \end{vmatrix}$$

$$= 2(-17) - 4(21 - 2) + 6(15 + 1)$$
$$= -34 - 4(19) + 6(16)$$
$$= -34 - 76 + 96$$
$$= -14$$ ✓ Same answer!

---

## 5. Choosing the Best Row/Column

### Strategy: Maximize Zeros

```
    | 3  0  0 |
    | 2  4  1 |     ← Expand along Row 1 (two zeros!)
    | 5  6  2 |
    
    det = 3·C₁₁ + 0·C₁₂ + 0·C₁₃
        = 3·C₁₁
        = 3 × |4 1|
              |6 2|
        = 3(8 - 6)
        = 6
```

### Comparison Table

| Expansion Choice | Number of 2×2 Dets to Calculate | Efficiency |
|-----------------|--------------------------------|------------|
| Row 1 (2 zeros) | 1 | ⭐⭐⭐ Best |
| Row 2 (0 zeros) | 3 | ⭐ Worst |
| Row 3 (0 zeros) | 3 | ⭐ Worst |
| Column 1 (0 zeros) | 3 | ⭐ Worst |
| Column 2 (0 zeros) | 3 | ⭐ Worst |
| Column 3 (0 zeros) | 3 | ⭐ Worst |

---

## 6. Creating Zeros Using Row Operations

### Powerful Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│                 ROW REDUCTION + EXPANSION                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   1. Use row/column operations to create zeros                  │
│   2. Expand along the row/column with zeros                     │
│   3. This dramatically reduces computation                       │
│                                                                  │
│   Remember: Adding a multiple of one row to another              │
│   does NOT change the determinant!                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Example with Row Operations

Evaluate: $\begin{vmatrix} 1 & 2 & 3 \\ 2 & 5 & 4 \\ 3 & 7 & 6 \end{vmatrix}$

**Step 1**: Create zeros in Column 1 using Row 1

$R_2 \to R_2 - 2R_1$:
$R_3 \to R_3 - 3R_1$:

$$\begin{vmatrix} 1 & 2 & 3 \\ 0 & 1 & -2 \\ 0 & 1 & -3 \end{vmatrix}$$

**Step 2**: Expand along Column 1

$$= 1 \cdot \begin{vmatrix} 1 & -2 \\ 1 & -3 \end{vmatrix} - 0 + 0$$

$$= 1(-3 - (-2)) = -3 + 2 = -1$$

---

## 7. Sarrus' Rule (3×3 Only)

### The Method

```
         a₁  a₂  a₃ | a₁  a₂
         b₁  b₂  b₃ | b₁  b₂
         c₁  c₂  c₃ | c₁  c₂
          ↘  ↘  ↘     ↗  ↗  ↗
          
    Positive diagonals     Negative diagonals
    (top-left to bottom)   (bottom-left to top)
    
    det = (a₁b₂c₃ + a₂b₃c₁ + a₃b₁c₂) - (c₁b₂a₃ + c₂b₃a₁ + c₃b₁a₂)
```

### Visual

```
    ┌─────────────────────────────────────────────────────────┐
    │                                                          │
    │    a₁──b₂──c₃  (+)    c₁──b₂──a₃  (-)                  │
    │       ╲  ╲                ╱  ╱                           │
    │    a₂──b₃──c₁  (+)    c₂──b₃──a₁  (-)                  │
    │       ╲  ╲                ╱  ╱                           │
    │    a₃──b₁──c₂  (+)    c₃──b₁──a₂  (-)                  │
    │                                                          │
    └─────────────────────────────────────────────────────────┘
```

### Example

$$\begin{vmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 7 & 8 & 9 \end{vmatrix}$$

Positive diagonals: $1 \cdot 5 \cdot 9 + 2 \cdot 6 \cdot 7 + 3 \cdot 4 \cdot 8 = 45 + 84 + 96 = 225$

Negative diagonals: $7 \cdot 5 \cdot 3 + 8 \cdot 6 \cdot 1 + 9 \cdot 4 \cdot 2 = 105 + 48 + 72 = 225$

$$\text{det} = 225 - 225 = 0$$

---

## 8. Upper Triangular Method

### The Most Efficient for Large Matrices

Transform to upper triangular form, then multiply diagonal elements.

### Procedure

```
┌─────────────────────────────────────────────────────────────────┐
│                TRIANGULAR REDUCTION METHOD                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Step 1: Start with first column                               │
│   Step 2: Use first row to eliminate elements below a₁₁         │
│   Step 3: Move to second column, use second row to eliminate    │
│   Step 4: Continue until upper triangular form achieved         │
│   Step 5: det = product of diagonal elements                    │
│                                                                  │
│   Note: Track sign changes if rows are swapped!                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Example

Evaluate: $\begin{vmatrix} 2 & 1 & 3 \\ 4 & 5 & 6 \\ 6 & 8 & 9 \end{vmatrix}$

**Step 1**: $R_2 \to R_2 - 2R_1$, $R_3 \to R_3 - 3R_1$

$$\begin{vmatrix} 2 & 1 & 3 \\ 0 & 3 & 0 \\ 0 & 5 & 0 \end{vmatrix}$$

**Step 2**: $R_3 \to R_3 - \frac{5}{3}R_2$

$$\begin{vmatrix} 2 & 1 & 3 \\ 0 & 3 & 0 \\ 0 & 0 & 0 \end{vmatrix}$$

**Result**: Upper triangular with a zero on diagonal.

$$\text{det} = 2 \times 3 \times 0 = 0$$

---

## 9. Block Matrix Determinant

### For Block Diagonal Matrices

$$\begin{vmatrix} A & O \\ O & B \end{vmatrix} = |A| \cdot |B|$$

### For Block Triangular Matrices

$$\begin{vmatrix} A & C \\ O & B \end{vmatrix} = |A| \cdot |B|$$

$$\begin{vmatrix} A & O \\ C & B \end{vmatrix} = |A| \cdot |B|$$

---

## 10. 4×4 Determinant Example

### Problem

Evaluate: $\begin{vmatrix} 1 & 2 & 0 & 0 \\ 3 & 4 & 0 & 0 \\ 0 & 0 & 5 & 6 \\ 0 & 0 & 7 & 8 \end{vmatrix}$

### Solution (Block Diagonal)

This is a block diagonal matrix:

$$|A| = \begin{vmatrix} 1 & 2 \\ 3 & 4 \end{vmatrix} \times \begin{vmatrix} 5 & 6 \\ 7 & 8 \end{vmatrix}$$

$$= (4 - 6)(40 - 42) = (-2)(-2) = 4$$

---

## 11. Expansion Method Comparison

| Method | Best For | Complexity |
|--------|----------|------------|
| Cofactor Expansion | Small matrices, matrices with zeros | O(n!) |
| Sarrus' Rule | 3×3 matrices only | O(1) |
| Row Reduction | Large matrices | O(n³) |
| Block Method | Block triangular/diagonal | Depends on blocks |

---

## 12. Common Mistakes to Avoid

```
┌─────────────────────────────────────────────────────────────────┐
│                     COMMON MISTAKES                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ✗ Forgetting the (-1)^(i+j) sign factor                       │
│                                                                  │
│   ✗ Not tracking sign changes when swapping rows                │
│                                                                  │
│   ✗ Using Sarrus' rule for matrices larger than 3×3             │
│                                                                  │
│   ✗ Forgetting to multiply by the scalar when factoring         │
│     a row out                                                    │
│                                                                  │
│   ✗ Making arithmetic errors in 2×2 determinants                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 13. Practice Problems

### Problem 1

Evaluate by expanding along Row 2:
$$\begin{vmatrix} 1 & 3 & 2 \\ 0 & 4 & 0 \\ 2 & 1 & 5 \end{vmatrix}$$

**Solution:**

$$= -0 \cdot C_{21} + 4 \cdot C_{22} - 0 \cdot C_{23}$$
$$= 4 \cdot \begin{vmatrix} 1 & 2 \\ 2 & 5 \end{vmatrix}$$
$$= 4(5 - 4) = 4$$

### Problem 2

Use row operations to evaluate:
$$\begin{vmatrix} 2 & 4 & 6 \\ 1 & 2 & 3 \\ 3 & 5 & 7 \end{vmatrix}$$

**Solution:**

Notice Row 1 = 2 × Row 2 (proportional rows)
$$\text{det} = 0$$

---

## 📊 Summary Table

| Method | When to Use | Key Formula |
|--------|-------------|-------------|
| Row Expansion | Any matrix | $\|A\| = \sum_j a_{ij}C_{ij}$ |
| Column Expansion | Columns with zeros | $\|A\| = \sum_i a_{ij}C_{ij}$ |
| Sarrus' Rule | 3×3 only | 3 positive - 3 negative diagonals |
| Row Reduction | Large matrices | Create upper triangular, multiply diagonal |
| Block Method | Block matrices | $\|A\| \cdot \|B\|$ |

---

## ❓ Quick Revision Questions

1. **Which row/column should you choose for expansion?**
   <details>
   <summary>Click for Answer</summary>
   The row or column with the most zeros, to minimize the number of cofactors you need to calculate.
   </details>

2. **What is the sign of the cofactor for position (2, 3)?**
   <details>
   <summary>Click for Answer</summary>
   $(-1)^{2+3} = (-1)^5 = -1$ (negative)
   </details>

3. **Does Sarrus' rule work for 4×4 matrices?**
   <details>
   <summary>Click for Answer</summary>
   No! Sarrus' rule only works for 3×3 matrices.
   </details>

4. **What happens to the determinant when you add 3×(Row 1) to Row 2?**
   <details>
   <summary>Click for Answer</summary>
   The determinant remains unchanged.
   </details>

5. **If you swap two rows, what happens to the determinant?**
   <details>
   <summary>Click for Answer</summary>
   The determinant changes sign (multiplied by -1).
   </details>

6. **For an upper triangular matrix, how do you find the determinant?**
   <details>
   <summary>Click for Answer</summary>
   Multiply all the diagonal elements together.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Minor and Cofactor](04-minor-and-cofactor.md) | [Unit 3: Determinants](./README.md) | [Product of Determinants →](../04-Determinant-Applications/01-product-of-determinants.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
