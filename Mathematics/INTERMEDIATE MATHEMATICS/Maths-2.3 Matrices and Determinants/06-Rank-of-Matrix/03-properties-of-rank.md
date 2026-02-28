# Chapter 6.3: Properties of Rank

[← Previous: Definition of Rank](02-definition-of-rank.md) | [Back to README](../README.md) | [Next: Rank and Systems →](04-rank-and-systems.md)

---

## 📚 Chapter Overview

Understanding the properties of rank helps in solving complex problems efficiently. This chapter explores how rank behaves under various matrix operations and establishes important inequalities.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Apply properties of rank in problem solving
- Use rank inequalities effectively
- Relate rank to other matrix properties
- Determine rank after matrix operations

---

## 1. Basic Properties

### Property 1: Rank Bounds

For an m×n matrix A:

$$0 \leq \text{rank}(A) \leq \min(m, n)$$

```
┌─────────────────────────────────────────────────────────────────┐
│                     RANK BOUNDS                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   m×n Matrix                                                     │
│                                                                  │
│       0 ≤ rank(A) ≤ min(m, n)                                   │
│                                                                  │
│   ┌───────────────┐                                             │
│   │               │ m rows                                       │
│   │               │                                              │
│   └───────────────┘                                             │
│         n columns                                                │
│                                                                  │
│   Examples:                                                      │
│   • 3×5 matrix: rank ≤ 3                                        │
│   • 5×3 matrix: rank ≤ 3                                        │
│   • 4×4 matrix: rank ≤ 4                                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Property 2: Zero Matrix

$$\text{rank}(O) = 0$$

The zero matrix is the only matrix with rank 0.

### Property 3: Rank of Transpose

$$\text{rank}(A^T) = \text{rank}(A)$$

Transpose doesn't change rank!

**Example**:
$$A = \begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \end{bmatrix}, \quad A^T = \begin{bmatrix} 1 & 4 \\ 2 & 5 \\ 3 & 6 \end{bmatrix}$$

Both have rank 2.

---

## 2. Rank and Elementary Operations

### Property 4: Elementary Row Operations Preserve Rank

Elementary row operations do NOT change the rank of a matrix.

| Operation | Effect on Rank |
|-----------|----------------|
| Swap rows | No change |
| Multiply row by non-zero scalar | No change |
| Add multiple of one row to another | No change |

This is WHY we can use row reduction to find rank!

### Property 5: Elementary Column Operations

Similarly, elementary column operations preserve rank.

---

## 3. Rank and Matrix Multiplication

### Property 6: Rank of Product

For matrices A (m×n) and B (n×p):

$$\text{rank}(AB) \leq \min(\text{rank}(A), \text{rank}(B))$$

**Interpretation**: Multiplying can only decrease (or maintain) rank, never increase it.

### Property 7: Product with Invertible Matrix

If P is an invertible m×m matrix and Q is an invertible n×n matrix:

$$\text{rank}(PA) = \text{rank}(A) = \text{rank}(AQ) = \text{rank}(PAQ)$$

**Example**:
$$A = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}, \quad P = \begin{bmatrix} 2 & 0 \\ 0 & 1 \end{bmatrix}$$

$$PA = \begin{bmatrix} 2 & 4 \\ 3 & 4 \end{bmatrix}$$

Both A and PA have rank 2.

---

## 4. Sylvester's Inequality

### Main Theorem

For matrices A (m×n) and B (n×p):

$$\text{rank}(A) + \text{rank}(B) - n \leq \text{rank}(AB) \leq \min(\text{rank}(A), \text{rank}(B))$$

```
┌─────────────────────────────────────────────────────────────────┐
│               SYLVESTER'S INEQUALITY                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   For A (m×n) and B (n×p):                                      │
│                                                                  │
│     rank(A) + rank(B) - n  ≤  rank(AB)  ≤  min(rank(A),rank(B)) │
│     ────────────────────      ────────      ──────────────────  │
│        Lower Bound            Product          Upper Bound       │
│                                Rank                              │
│                                                                  │
│   Note: n = number of columns of A = number of rows of B        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Example

A is 3×4 with rank 3, B is 4×2 with rank 2.

Lower bound: $3 + 2 - 4 = 1$
Upper bound: $\min(3, 2) = 2$

So: $1 \leq \text{rank}(AB) \leq 2$

---

## 5. Rank of Sum

### Property 8: Subadditivity

$$\text{rank}(A + B) \leq \text{rank}(A) + \text{rank}(B)$$

### Property 9: Frobenius Inequality

$$\text{rank}(A + B) \geq |\text{rank}(A) - \text{rank}(B)|$$

Combining both:
$$|\text{rank}(A) - \text{rank}(B)| \leq \text{rank}(A + B) \leq \text{rank}(A) + \text{rank}(B)$$

### Example

$$A = \begin{bmatrix} 1 & 0 \\ 0 & 0 \end{bmatrix}, \quad B = \begin{bmatrix} 0 & 0 \\ 0 & 1 \end{bmatrix}$$

- rank(A) = 1, rank(B) = 1
- $A + B = \begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix}$
- rank(A + B) = 2

Check: $|1 - 1| \leq 2 \leq 1 + 1$ ✓

---

## 6. Rank and Invertibility

### Property 10: Square Matrix Invertibility

For an n×n matrix A:

$$A \text{ is invertible} \iff \text{rank}(A) = n \iff |A| \neq 0$$

### Property 11: Product Invertibility

If A and B are both n×n and invertible:
- rank(A) = rank(B) = n
- rank(AB) = n
- AB is also invertible

---

## 7. Rank and Powers

### Property 12: Powers of a Matrix

For any square matrix A:

$$\text{rank}(A^2) \leq \text{rank}(A)$$

More generally:
$$\text{rank}(A^{k+1}) \leq \text{rank}(A^k)$$

**Special Case**: If $A^k = O$ (nilpotent matrix), then rank decreases with each power.

### Example

$$A = \begin{bmatrix} 0 & 1 \\ 0 & 0 \end{bmatrix}$$

- rank(A) = 1
- $A^2 = \begin{bmatrix} 0 & 0 \\ 0 & 0 \end{bmatrix}$
- rank(A²) = 0

---

## 8. Rank of Block Matrices

### Block Diagonal

$$\text{rank}\begin{pmatrix} A & O \\ O & B \end{pmatrix} = \text{rank}(A) + \text{rank}(B)$$

### General Block Matrix

$$\text{rank}\begin{pmatrix} A & B \\ C & D \end{pmatrix} \geq \text{rank}(A)$$

---

## 9. Rank-Nullity Theorem

### Definition of Nullity

The **nullity** of an m×n matrix A is the dimension of its null space (solution space of Ax = 0).

$$\text{nullity}(A) = n - \text{rank}(A)$$

Or equivalently:

$$\text{rank}(A) + \text{nullity}(A) = n$$

```
┌─────────────────────────────────────────────────────────────────┐
│               RANK-NULLITY THEOREM                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   For an m×n matrix A:                                          │
│                                                                  │
│        rank(A) + nullity(A) = n                                 │
│        ───────   ──────────   ─                                 │
│        # pivot   # free       # columns                         │
│        columns   variables                                       │
│                                                                  │
│   Example: A is 3×5 with rank 2                                 │
│            nullity(A) = 5 - 2 = 3                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 10. Important Worked Examples

### Example 1: Find rank after operations

Given rank(A) = 3 for 3×3 matrix A. Find rank(2A).

**Solution**:
Multiplying by scalar 2 is equivalent to multiplying each row by 2.
This doesn't change rank.

**rank(2A) = 3**

### Example 2: Rank of product

If A is 3×4 with rank 3, and B is 4×5 with rank 4, find possible values of rank(AB).

**Solution**:
Using Sylvester's inequality:
- Lower bound: $3 + 4 - 4 = 3$
- Upper bound: $\min(3, 4) = 3$

**rank(AB) = 3**

### Example 3: Rank comparison

If A² = A (idempotent), what can you say about rank(A)?

**Solution**:
For idempotent matrices, rank(A) = trace(A) (sum of eigenvalues = sum of diagonal elements).

Also, rank(A²) = rank(A) (since A² = A).

This is consistent with the property that rank(A²) ≤ rank(A).

---

## 📊 Properties Summary Table

| Property | Statement |
|----------|-----------|
| Bound | $0 \leq \text{rank}(A) \leq \min(m, n)$ |
| Transpose | $\text{rank}(A^T) = \text{rank}(A)$ |
| Product (upper) | $\text{rank}(AB) \leq \min(\text{rank}(A), \text{rank}(B))$ |
| Sylvester (lower) | $\text{rank}(AB) \geq \text{rank}(A) + \text{rank}(B) - n$ |
| Sum (upper) | $\text{rank}(A + B) \leq \text{rank}(A) + \text{rank}(B)$ |
| Invertible product | $\text{rank}(PA) = \text{rank}(AQ) = \text{rank}(A)$ |
| Powers | $\text{rank}(A^{k+1}) \leq \text{rank}(A^k)$ |
| Rank-Nullity | $\text{rank}(A) + \text{nullity}(A) = n$ |

---

## ❓ Quick Revision Questions

1. **If A is 4×6, what is the maximum possible rank?**
   <details>
   <summary>Click for Answer</summary>
   4 (minimum of 4 and 6)
   </details>

2. **If rank(A) = 3, what is rank(Aᵀ)?**
   <details>
   <summary>Click for Answer</summary>
   3 (transpose preserves rank)
   </details>

3. **Can multiplying two rank-2 matrices give a rank-3 product?**
   <details>
   <summary>Click for Answer</summary>
   No. rank(AB) ≤ min(rank(A), rank(B)) ≤ 2
   </details>

4. **If A is 5×5 with rank 5, is A invertible?**
   <details>
   <summary>Click for Answer</summary>
   Yes (full rank for square matrix implies invertibility)
   </details>

5. **A is 3×7 with rank 3. What is nullity(A)?**
   <details>
   <summary>Click for Answer</summary>
   nullity(A) = 7 - 3 = 4
   </details>

6. **Do elementary row operations change rank?**
   <details>
   <summary>Click for Answer</summary>
   No, they preserve rank.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Definition of Rank](02-definition-of-rank.md) | [Unit 6: Rank](./README.md) | [Rank and Systems →](04-rank-and-systems.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
