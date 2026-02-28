# Chapter 4.1: Product of Determinants

[← Previous: Expansion Methods](../03-Determinants/05-expansion-methods.md) | [Back to README](../README.md) | [Next: Determinant of Transpose →](02-determinant-of-transpose.md)

---

## 📚 Chapter Overview

This chapter explores the multiplicative property of determinants—one of the most powerful and useful properties. Understanding how determinants behave under matrix multiplication opens doors to solving complex problems efficiently.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- State and apply the product theorem of determinants
- Prove the product property for specific cases
- Apply the property to simplify calculations
- Extend the property to powers and inverses

---

## 1. The Product Theorem

### Statement

> For two square matrices A and B of the same order:
> $$|AB| = |A| \cdot |B|$$

### In Words

The determinant of a product equals the product of the determinants.

### Visual Representation

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRODUCT THEOREM                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│     Matrix A          Matrix B           Matrix AB              │
│    ┌───────┐         ┌───────┐         ┌───────┐               │
│    │       │    ×    │       │    =    │       │               │
│    │   A   │         │   B   │         │  AB   │               │
│    │       │         │       │         │       │               │
│    └───────┘         └───────┘         └───────┘               │
│       │                 │                  │                    │
│       │ det             │ det              │ det                │
│       ▼                 ▼                  ▼                    │
│      |A|       ×       |B|       =       |AB|                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Numerical Verification

### Example 1: 2×2 Matrices

Let:
$$A = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}, \quad B = \begin{bmatrix} 5 & 6 \\ 7 & 8 \end{bmatrix}$$

**Step 1**: Calculate individual determinants

$$|A| = (1)(4) - (2)(3) = 4 - 6 = -2$$
$$|B| = (5)(8) - (6)(7) = 40 - 42 = -2$$

**Step 2**: Calculate the product |A| · |B|

$$|A| \cdot |B| = (-2)(-2) = 4$$

**Step 3**: Calculate AB

$$AB = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}\begin{bmatrix} 5 & 6 \\ 7 & 8 \end{bmatrix}$$

$$= \begin{bmatrix} 1(5)+2(7) & 1(6)+2(8) \\ 3(5)+4(7) & 3(6)+4(8) \end{bmatrix}$$

$$= \begin{bmatrix} 19 & 22 \\ 43 & 50 \end{bmatrix}$$

**Step 4**: Calculate |AB|

$$|AB| = (19)(50) - (22)(43) = 950 - 946 = 4$$

**Verification**: $|AB| = 4 = |A| \cdot |B|$ ✓

---

## 3. Example 2: 3×3 Matrices

Let:
$$A = \begin{bmatrix} 1 & 0 & 0 \\ 0 & 2 & 0 \\ 0 & 0 & 3 \end{bmatrix}, \quad B = \begin{bmatrix} 1 & 2 & 3 \\ 0 & 1 & 4 \\ 0 & 0 & 1 \end{bmatrix}$$

**Determinants**:
- $|A| = 1 \cdot 2 \cdot 3 = 6$ (diagonal matrix)
- $|B| = 1 \cdot 1 \cdot 1 = 1$ (upper triangular)

**Product**:
$$|A| \cdot |B| = 6 \cdot 1 = 6$$

**Calculate AB**:
$$AB = \begin{bmatrix} 1 & 2 & 3 \\ 0 & 2 & 8 \\ 0 & 0 & 3 \end{bmatrix}$$

**Determinant of AB**: $|AB| = 1 \cdot 2 \cdot 3 = 6$ ✓

---

## 4. Important Corollaries

### Corollary 1: Power of a Matrix

$$|A^n| = |A|^n$$

**Proof**:
$$|A^2| = |A \cdot A| = |A| \cdot |A| = |A|^2$$
$$|A^3| = |A^2 \cdot A| = |A|^2 \cdot |A| = |A|^3$$

By induction: $|A^n| = |A|^n$

### Corollary 2: Inverse Matrix

If A is invertible (i.e., $|A| \neq 0$), then:
$$|A^{-1}| = \frac{1}{|A|}$$

**Proof**:
$$AA^{-1} = I$$
$$|AA^{-1}| = |I|$$
$$|A| \cdot |A^{-1}| = 1$$
$$|A^{-1}| = \frac{1}{|A|}$$

### Corollary 3: Product of Inverses

$$|(AB)^{-1}| = |A^{-1}| \cdot |B^{-1}| = \frac{1}{|A| \cdot |B|}$$

---

## 5. Corollaries Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                    KEY COROLLARIES                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   1. |A^n| = |A|^n                                              │
│                                                                  │
│   2. |A^(-1)| = 1/|A|        (if A is invertible)              │
│                                                                  │
│   3. |A^(-n)| = 1/|A|^n                                         │
│                                                                  │
│   4. |kA| = k^n|A|           (for n×n matrix A)                 │
│                                                                  │
│   5. |A||A^(-1)| = 1                                            │
│                                                                  │
│   6. If |A| = 0, then A^(-1) does not exist                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Order Does Not Matter

### Important Note

Since multiplication of determinants (numbers) is commutative:
$$|A| \cdot |B| = |B| \cdot |A|$$

Therefore:
$$|AB| = |BA|$$

**Even though $AB \neq BA$ in general!**

### Example

$$A = \begin{bmatrix} 1 & 2 \\ 0 & 3 \end{bmatrix}, \quad B = \begin{bmatrix} 4 & 1 \\ 2 & 0 \end{bmatrix}$$

$$|A| = 3, \quad |B| = -2$$

$$|AB| = |A| \cdot |B| = 3 \times (-2) = -6$$
$$|BA| = |B| \cdot |A| = (-2) \times 3 = -6$$

Both equal -6, even though $AB \neq BA$.

---

## 7. Extension to Multiple Matrices

### Three or More Matrices

$$|ABC| = |A| \cdot |B| \cdot |C|$$

More generally:
$$|A_1 A_2 \cdots A_n| = |A_1| \cdot |A_2| \cdots |A_n|$$

### Example

If $|A| = 2$, $|B| = 3$, $|C| = 5$:
$$|ABC| = 2 \times 3 \times 5 = 30$$

---

## 8. Application: Checking Invertibility

### When is AB Invertible?

$$AB \text{ is invertible} \Leftrightarrow |AB| \neq 0$$
$$\Leftrightarrow |A| \cdot |B| \neq 0$$
$$\Leftrightarrow |A| \neq 0 \text{ AND } |B| \neq 0$$

### Conclusion

> The product of two matrices is invertible if and only if BOTH matrices are invertible.

```
┌─────────────────────────────────────────────────────────────────┐
│                 INVERTIBILITY CHECK                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   AB invertible ←→ A invertible AND B invertible                │
│                                                                  │
│   If either |A| = 0 or |B| = 0:                                 │
│   → |AB| = 0                                                    │
│   → AB is NOT invertible (singular)                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Special Cases

### Case 1: One Matrix is Singular

If $|A| = 0$:
$$|AB| = |A| \cdot |B| = 0 \cdot |B| = 0$$

AB is also singular.

### Case 2: Identity Matrix

$$|AI| = |A| \cdot |I| = |A| \cdot 1 = |A|$$

### Case 3: Scalar Matrix

$$|A(kI)| = |A| \cdot |kI| = |A| \cdot k^n$$

---

## 10. Proof Outline (Conceptual)

### Why Does $|AB| = |A| \cdot |B|$?

**Geometric Interpretation**:
- The determinant represents the "scaling factor" for volume
- Matrix A scales volume by factor |A|
- Matrix B scales volume by factor |B|
- The combined transformation AB scales by |A| × |B|

**Algebraic Approach** (for 2×2):

Let $A = \begin{bmatrix} a & b \\ c & d \end{bmatrix}$ and $B = \begin{bmatrix} e & f \\ g & h \end{bmatrix}$

$$AB = \begin{bmatrix} ae+bg & af+bh \\ ce+dg & cf+dh \end{bmatrix}$$

$$|AB| = (ae+bg)(cf+dh) - (af+bh)(ce+dg)$$

Expanding and simplifying (tedious but straightforward):
$$= (ad-bc)(eh-fg) = |A| \cdot |B|$$

---

## 11. Practice Problems

### Problem 1

If $|A| = 3$ and $|B| = -2$, find $|A^2B^3|$.

**Solution**:
$$|A^2B^3| = |A^2| \cdot |B^3| = |A|^2 \cdot |B|^3$$
$$= 3^2 \cdot (-2)^3 = 9 \cdot (-8) = -72$$

### Problem 2

If $|A| = 4$ and $|B| = 2$, find $|(AB)^{-1}|$.

**Solution**:
$$|(AB)^{-1}| = \frac{1}{|AB|} = \frac{1}{|A| \cdot |B|} = \frac{1}{4 \times 2} = \frac{1}{8}$$

### Problem 3

If $A$ is a 3×3 matrix with $|A| = 2$, find $|3A^2|$.

**Solution**:
$$|3A^2| = 3^3 \cdot |A^2| = 27 \cdot |A|^2 = 27 \cdot 4 = 108$$

---

## 📊 Summary Table

| Property | Formula | Condition |
|----------|---------|-----------|
| Product of matrices | $\|AB\| = \|A\| \cdot \|B\|$ | A, B are n×n |
| Power of matrix | $\|A^n\| = \|A\|^n$ | n ≥ 1 |
| Inverse | $\|A^{-1}\| = 1/\|A\|$ | $\|A\| \neq 0$ |
| Multiple matrices | $\|A_1A_2...A_k\| = \|A_1\|\|A_2\|...\|A_k\|$ | All same order |
| Scalar multiple | $\|kA\| = k^n\|A\|$ | A is n×n |
| Invertibility | AB invertible iff both invertible | |

---

## ❓ Quick Revision Questions

1. **If $|A| = 5$ and $|B| = 3$, what is $|AB|$?**
   <details>
   <summary>Click for Answer</summary>
   $|AB| = |A| \cdot |B| = 5 \times 3 = 15$
   </details>

2. **If $|A| = 4$, what is $|A^{-1}|$?**
   <details>
   <summary>Click for Answer</summary>
   $|A^{-1}| = 1/|A| = 1/4$
   </details>

3. **If $|A| = 2$, what is $|A^5|$?**
   <details>
   <summary>Click for Answer</summary>
   $|A^5| = |A|^5 = 2^5 = 32$
   </details>

4. **If $|A| = 0$, is AB invertible?**
   <details>
   <summary>Click for Answer</summary>
   No. $|AB| = |A| \cdot |B| = 0 \cdot |B| = 0$, so AB is singular.
   </details>

5. **Is $|AB| = |BA|$ always true?**
   <details>
   <summary>Click for Answer</summary>
   Yes! $|AB| = |A||B| = |B||A| = |BA|$, even though $AB \neq BA$ in general.
   </details>

6. **If A is 3×3 with $|A| = 2$, find $|2A|$.**
   <details>
   <summary>Click for Answer</summary>
   $|2A| = 2^3 \cdot |A| = 8 \times 2 = 16$
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Expansion Methods](../03-Determinants/05-expansion-methods.md) | [Unit 4: Applications](./README.md) | [Determinant of Transpose →](02-determinant-of-transpose.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
