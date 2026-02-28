# Chapter 1.4: Equality of Matrices

[← Previous: Order of Matrix](03-order-of-matrix.md) | [Back to README](../README.md) | [Next: Unit 2 - Addition and Subtraction →](../02-Matrix-Operations/01-addition-and-subtraction.md)

---

## 📚 Chapter Overview

Understanding when two matrices are considered equal is fundamental for solving matrix equations and verifying matrix operations. This chapter establishes the precise conditions for matrix equality.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- State the conditions for matrix equality
- Determine if two matrices are equal
- Solve equations using matrix equality
- Apply equality conditions to find unknown elements

---

## 1. Definition of Matrix Equality

### Formal Definition

> Two matrices A and B are **equal** (written A = B) if and only if:
> 1. They have the **same order** (same number of rows and columns)
> 2. **Corresponding elements** are equal (elements in the same position are identical)

### Mathematical Statement

If $A = [a_{ij}]_{m \times n}$ and $B = [b_{ij}]_{p \times q}$, then:

$$A = B \iff \begin{cases} m = p \text{ (same rows)} \\ n = q \text{ (same columns)} \\ a_{ij} = b_{ij} \text{ for all } i, j \end{cases}$$

---

## 2. Visual Understanding

### Condition 1: Same Order

```
        Matrix A (2×3)              Matrix B (2×3)
        
        ┌───────────────┐          ┌───────────────┐
        │  1   2   3    │          │  1   2   3    │
        │  4   5   6    │          │  4   5   6    │
        └───────────────┘          └───────────────┘
        
        Same order? ✓ (Both are 2×3)
```

```
        Matrix A (2×3)              Matrix C (3×2)
        
        ┌───────────────┐          ┌─────────┐
        │  1   2   3    │          │  1   2  │
        │  4   5   6    │          │  3   4  │
        └───────────────┘          │  5   6  │
                                   └─────────┘
        
        Same order? ✗ (2×3 ≠ 3×2)
        Therefore A ≠ C (regardless of elements)
```

### Condition 2: Corresponding Elements Equal

```
        Matrix A                    Matrix B
        
        ┌─────────────────┐        ┌─────────────────┐
        │  a₁₁  a₁₂  a₁₃ │        │  b₁₁  b₁₂  b₁₃ │
        │  a₂₁  a₂₂  a₂₃ │        │  b₂₁  b₂₂  b₂₃ │
        └─────────────────┘        └─────────────────┘
        
        For A = B, we need:
        
        a₁₁ = b₁₁  ←→  a₁₂ = b₁₂  ←→  a₁₃ = b₁₃
           ↕              ↕              ↕
        a₂₁ = b₂₁  ←→  a₂₂ = b₂₂  ←→  a₂₃ = b₂₃
        
        ALL corresponding elements must match!
```

---

## 3. Examples of Equal and Unequal Matrices

### Example 1: Equal Matrices

$$A = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix} \quad B = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}$$

**Analysis**:
- Order of A: 2×2 ✓
- Order of B: 2×2 ✓
- $a_{11} = b_{11} = 1$ ✓
- $a_{12} = b_{12} = 2$ ✓
- $a_{21} = b_{21} = 3$ ✓
- $a_{22} = b_{22} = 4$ ✓

**Conclusion**: A = B ✓

### Example 2: Different Orders

$$A = \begin{bmatrix} 1 & 2 & 3 \end{bmatrix} \quad B = \begin{bmatrix} 1 \\ 2 \\ 3 \end{bmatrix}$$

**Analysis**:
- Order of A: 1×3 (row matrix)
- Order of B: 3×1 (column matrix)
- Orders are different

**Conclusion**: A ≠ B ✗

### Example 3: Same Order, Different Elements

$$A = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix} \quad B = \begin{bmatrix} 1 & 2 \\ 3 & 5 \end{bmatrix}$$

**Analysis**:
- Both are 2×2 ✓
- $a_{22} = 4$ but $b_{22} = 5$
- One element differs

**Conclusion**: A ≠ B ✗

### Example 4: Equivalent Expressions

$$A = \begin{bmatrix} 2+3 & 6/2 \\ \sqrt{9} & 2^2 \end{bmatrix} \quad B = \begin{bmatrix} 5 & 3 \\ 3 & 4 \end{bmatrix}$$

**Analysis**:
- Both are 2×2 ✓
- $a_{11} = 2+3 = 5 = b_{11}$ ✓
- $a_{12} = 6/2 = 3 = b_{12}$ ✓
- $a_{21} = \sqrt{9} = 3 = b_{21}$ ✓
- $a_{22} = 2^2 = 4 = b_{22}$ ✓

**Conclusion**: A = B ✓

---

## 4. Solving Equations Using Matrix Equality

### Method

When two matrices are equal, equate corresponding elements to form a system of equations.

### Example 1: Finding Unknown Values

**Problem**: Find x and y if:

$$\begin{bmatrix} x+2 & 5 \\ 7 & y-3 \end{bmatrix} = \begin{bmatrix} 6 & 5 \\ 7 & 4 \end{bmatrix}$$

**Solution**:

```
Step 1: Verify same order
        Both matrices are 2×2 ✓

Step 2: Equate corresponding elements
        
        Position (1,1): x + 2 = 6
        Position (1,2): 5 = 5      (already satisfied)
        Position (2,1): 7 = 7      (already satisfied)
        Position (2,2): y - 3 = 4

Step 3: Solve equations
        x + 2 = 6  →  x = 4
        y - 3 = 4  →  y = 7
```

**Answer**: x = 4, y = 7

### Example 2: System of Equations

**Problem**: Find a, b, c if:

$$\begin{bmatrix} a+b & 2a-b \\ c+1 & 3c \end{bmatrix} = \begin{bmatrix} 5 & 1 \\ 4 & 9 \end{bmatrix}$$

**Solution**:

```
Step 1: Set up equations from corresponding elements
        
        a + b = 5    ... (1)
        2a - b = 1   ... (2)
        c + 1 = 4    ... (3)
        3c = 9       ... (4)

Step 2: Solve for c (equations 3 and 4)
        From (4): c = 3
        Check in (3): 3 + 1 = 4 ✓

Step 3: Solve for a and b (equations 1 and 2)
        Adding (1) and (2):
        (a + b) + (2a - b) = 5 + 1
        3a = 6
        a = 2
        
        Substituting in (1):
        2 + b = 5
        b = 3
```

**Answer**: a = 2, b = 3, c = 3

### Example 3: More Complex Problem

**Problem**: Find x, y, z, w if:

$$\begin{bmatrix} x+y & 2z+w \\ x-y & z-w \end{bmatrix} = \begin{bmatrix} 4 & 7 \\ 2 & 1 \end{bmatrix}$$

**Solution**:

```
        Equations:
        x + y = 4    ... (1)
        x - y = 2    ... (2)
        2z + w = 7   ... (3)
        z - w = 1    ... (4)

        From (1) and (2):
        Adding: 2x = 6 → x = 3
        Subtracting: 2y = 2 → y = 1

        From (3) and (4):
        From (4): z = 1 + w
        Substituting in (3): 2(1 + w) + w = 7
                            2 + 2w + w = 7
                            3w = 5
                            w = 5/3
        
        z = 1 + 5/3 = 8/3
```

**Answer**: x = 3, y = 1, z = 8/3, w = 5/3

---

## 5. Special Cases

### Case 1: Matrices with Functions

$$\begin{bmatrix} \sin\theta & \cos\theta \\ 0 & 1 \end{bmatrix} = \begin{bmatrix} 0.5 & \frac{\sqrt{3}}{2} \\ 0 & 1 \end{bmatrix}$$

From $\sin\theta = 0.5$, we get $\theta = 30°$ or $\theta = 150°$

Check: $\cos 30° = \frac{\sqrt{3}}{2}$ ✓

**Answer**: $\theta = 30°$

### Case 2: No Solution

$$\begin{bmatrix} x & 3 \\ 4 & 5 \end{bmatrix} = \begin{bmatrix} 2 & 3 \\ 4 & x \end{bmatrix}$$

From position (1,1): x = 2
From position (2,2): 5 = x → x = 5

**Contradiction**: x cannot be both 2 and 5!

**Answer**: No solution exists

### Case 3: Infinitely Many Solutions

$$\begin{bmatrix} x+y & 5 \\ 7 & 9 \end{bmatrix} = \begin{bmatrix} 10 & 5 \\ 7 & 9 \end{bmatrix}$$

Only condition: x + y = 10

Any values of x and y satisfying this work:
- x = 5, y = 5
- x = 7, y = 3
- x = 0, y = 10
- etc.

**Answer**: Infinitely many solutions where x + y = 10

---

## 6. Decision Flowchart

```
┌─────────────────────────────────────────────────────────────────┐
│               CHECKING MATRIX EQUALITY                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                    ┌─────────────────┐                          │
│                    │  Given A and B  │                          │
│                    └────────┬────────┘                          │
│                             │                                    │
│                             ▼                                    │
│              ┌──────────────────────────────┐                   │
│              │  Are the orders the same?    │                   │
│              │  (rows_A = rows_B AND        │                   │
│              │   cols_A = cols_B)           │                   │
│              └──────────────┬───────────────┘                   │
│                             │                                    │
│                    ┌────────┴────────┐                          │
│                    │                 │                          │
│                   YES               NO                          │
│                    │                 │                          │
│                    ▼                 ▼                          │
│    ┌───────────────────────┐   ┌─────────────┐                 │
│    │ Check ALL elements:   │   │  A ≠ B      │                 │
│    │ a_ij = b_ij for all   │   │  (UNEQUAL)  │                 │
│    │ positions (i,j)?      │   └─────────────┘                 │
│    └───────────┬───────────┘                                   │
│                │                                                │
│       ┌────────┴────────┐                                      │
│       │                 │                                      │
│      YES               NO                                      │
│       │                 │                                      │
│       ▼                 ▼                                      │
│  ┌─────────┐      ┌─────────────┐                              │
│  │  A = B  │      │   A ≠ B     │                              │
│  │ (EQUAL) │      │  (UNEQUAL)  │                              │
│  └─────────┘      └─────────────┘                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Common Mistakes to Avoid

### Mistake 1: Ignoring Order

```
        WRONG:
        [1 2 3] = [1]    ← Different orders!
                  [2]
                  [3]
        
        These are NOT equal despite having same elements!
```

### Mistake 2: Partial Checking

```
        WRONG approach:
        "The first elements match, so matrices are equal"
        
        CORRECT approach:
        ALL corresponding elements must be checked
```

### Mistake 3: Confusing Position

```
        A = [1  2]      B = [1  3]
            [3  4]          [2  4]
        
        a₁₂ = 2  but  b₁₂ = 3  → A ≠ B
        
        Even though both contain 1, 2, 3, 4
```

---

## 📊 Summary Table

| Condition | Requirement | What to Check |
|-----------|-------------|---------------|
| Same Order | m₁ = m₂ and n₁ = n₂ | Count rows and columns |
| Element Equality | aᵢⱼ = bᵢⱼ for all i, j | Compare each position |
| Solving for unknowns | Set up equations | Equate corresponding elements |
| No solution | Contradictory equations | Check consistency |
| Infinite solutions | Dependent equations | Identify free variables |

### Key Formulas

| Scenario | Equations Generated |
|----------|---------------------|
| 2×2 matrices | 4 equations |
| 3×3 matrices | 9 equations |
| m×n matrices | m×n equations |

---

## ❓ Quick Revision Questions

1. **What are the two conditions for matrix equality?**
   <details>
   <summary>Click for Answer</summary>
   1. Both matrices must have the same order (same dimensions)
   2. All corresponding elements must be equal (aᵢⱼ = bᵢⱼ for all i, j)
   </details>

2. **Find x if $\begin{bmatrix} 2x+1 & 5 \\ 3 & 7 \end{bmatrix} = \begin{bmatrix} 9 & 5 \\ 3 & 7 \end{bmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   From position (1,1): 2x + 1 = 9
   2x = 8
   x = 4
   </details>

3. **Are $\begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}$ and $\begin{bmatrix} 1 & 3 \\ 2 & 4 \end{bmatrix}$ equal?**
   <details>
   <summary>Click for Answer</summary>
   No. While they have the same order (2×2), the elements at position (1,2) and (2,1) are different.
   A has a₁₂=2, a₂₁=3
   B has b₁₂=3, b₂₁=2
   </details>

4. **Find a and b if $\begin{bmatrix} a-b & 5 \\ 4 & a+b \end{bmatrix} = \begin{bmatrix} 3 & 5 \\ 4 & 7 \end{bmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   a - b = 3 ... (1)
   a + b = 7 ... (2)
   Adding: 2a = 10 → a = 5
   From (2): 5 + b = 7 → b = 2
   Answer: a = 5, b = 2
   </details>

5. **Is $\begin{bmatrix} \sqrt{4} & 3 \\ 2 & 5 \end{bmatrix} = \begin{bmatrix} 2 & 3 \\ 2 & 5 \end{bmatrix}$?**
   <details>
   <summary>Click for Answer</summary>
   Yes! √4 = 2, so all corresponding elements are equal and the orders match (both 2×2).
   </details>

6. **How many equations are generated when comparing two 3×4 matrices for equality?**
   <details>
   <summary>Click for Answer</summary>
   3 × 4 = 12 equations (one for each element position)
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Order of Matrix](03-order-of-matrix.md) | [Unit 1: Introduction](./README.md) | [Unit 2: Addition and Subtraction →](../02-Matrix-Operations/01-addition-and-subtraction.md) |

---

**🎉 Congratulations!** You have completed **Unit 1: Introduction to Matrices**

### Unit 1 Checklist:
- [x] Definition and Notation
- [x] Types of Matrices
- [x] Order of a Matrix
- [x] Equality of Matrices

**[Continue to Unit 2: Matrix Operations →](../02-Matrix-Operations/01-addition-and-subtraction.md)**

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
