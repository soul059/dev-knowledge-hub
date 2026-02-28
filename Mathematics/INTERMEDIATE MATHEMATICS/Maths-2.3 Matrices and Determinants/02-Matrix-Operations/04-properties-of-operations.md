# Chapter 2.4: Properties of Matrix Operations

[← Previous: Matrix Multiplication](03-matrix-multiplication.md) | [Back to README](../README.md) | [Next: Transpose of Matrix →](05-transpose-of-matrix.md)

---

## 📚 Chapter Overview

This chapter consolidates all the properties of matrix operations including addition, scalar multiplication, and matrix multiplication. Understanding these properties is essential for matrix algebra and solving complex problems.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- State and apply all matrix operation properties
- Distinguish between commutative and non-commutative operations
- Use properties to simplify matrix expressions
- Understand the algebraic structure of matrices

---

## 1. Properties of Matrix Addition

### Complete List

| Property | Statement | Formula |
|----------|-----------|---------|
| Closure | Sum of two m×n matrices is m×n | $A_{m×n} + B_{m×n} = C_{m×n}$ |
| Commutative | Order doesn't matter | $A + B = B + A$ |
| Associative | Grouping doesn't matter | $(A + B) + C = A + (B + C)$ |
| Additive Identity | Zero matrix exists | $A + O = O + A = A$ |
| Additive Inverse | Negative matrix exists | $A + (-A) = O$ |

### Visual Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                    ADDITION PROPERTIES                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   COMMUTATIVE           ASSOCIATIVE          IDENTITY            │
│   ───────────           ───────────          ────────            │
│                                                                  │
│     A + B               (A + B) + C          A + O               │
│       ‖                     ‖                  ‖                 │
│     B + A               A + (B + C)            A                 │
│                                                                  │
│                                                                  │
│   INVERSE                                                        │
│   ───────                                                        │
│                                                                  │
│     A + (-A) = O         where (-A)ᵢⱼ = -aᵢⱼ                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Properties of Scalar Multiplication

### Complete List

| Property | Statement | Formula |
|----------|-----------|---------|
| Closure | kA is same order as A | $k \cdot A_{m×n} = B_{m×n}$ |
| Associative | Scalars group freely | $(kl)A = k(lA)$ |
| Distributive (scalar) | Scalars distribute | $(k + l)A = kA + lA$ |
| Distributive (matrix) | Matrix distributes | $k(A + B) = kA + kB$ |
| Multiplicative Identity | 1 preserves matrix | $1 \cdot A = A$ |
| Zero Scalar | 0 gives zero matrix | $0 \cdot A = O$ |

### Property Relationships

```
┌─────────────────────────────────────────────────────────────────┐
│               SCALAR MULTIPLICATION PROPERTIES                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│         Distributive Laws                                        │
│         ─────────────────                                        │
│                                                                  │
│    ┌───────────────────────────────────────────┐                │
│    │                                            │                │
│    │   k(A + B) = kA + kB                      │                │
│    │      ↑                                    │                │
│    │   Scalar distributes over matrix sum     │                │
│    │                                            │                │
│    └───────────────────────────────────────────┘                │
│                                                                  │
│    ┌───────────────────────────────────────────┐                │
│    │                                            │                │
│    │   (k + l)A = kA + lA                      │                │
│    │      ↑                                    │                │
│    │   Scalar sum distributes over matrix     │                │
│    │                                            │                │
│    └───────────────────────────────────────────┘                │
│                                                                  │
│         Associative Law                                          │
│         ───────────────                                          │
│                                                                  │
│    ┌───────────────────────────────────────────┐                │
│    │                                            │                │
│    │   (kl)A = k(lA) = l(kA)                   │                │
│    │      ↑                                    │                │
│    │   Order of scalar multiplication free    │                │
│    │                                            │                │
│    └───────────────────────────────────────────┘                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Properties of Matrix Multiplication

### Complete List

| Property | Statement | Formula |
|----------|-----------|---------|
| Closure | Product has predictable order | $(m×n)(n×p) = (m×p)$ |
| Associative | Grouping doesn't matter | $(AB)C = A(BC)$ |
| **NOT Commutative** | Order DOES matter | $AB \neq BA$ (generally) |
| Left Distributive | Distributes over addition | $A(B + C) = AB + AC$ |
| Right Distributive | Distributes over addition | $(A + B)C = AC + BC$ |
| Scalar Compatible | Scalars can move freely | $k(AB) = (kA)B = A(kB)$ |
| Multiplicative Identity | Identity matrix exists | $AI = IA = A$ |
| Zero Property | Zero absorbs | $AO = OA = O$ |

### Critical: Non-Commutativity

```
┌─────────────────────────────────────────────────────────────────┐
│                 ⚠️  WARNING: NON-COMMUTATIVE  ⚠️                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│     Matrix multiplication is NOT commutative!                    │
│                                                                  │
│     AB ≠ BA  (in general)                                       │
│                                                                  │
│     ┌─────────────────────────────────────────────────────┐     │
│     │  Case 1: Only one product exists                    │     │
│     │          A(2×3) × B(3×4) exists                     │     │
│     │          B(3×4) × A(2×3) does NOT exist             │     │
│     └─────────────────────────────────────────────────────┘     │
│                                                                  │
│     ┌─────────────────────────────────────────────────────┐     │
│     │  Case 2: Both exist but different sizes             │     │
│     │          A(2×3) × B(3×2) = 2×2                      │     │
│     │          B(3×2) × A(2×3) = 3×3                      │     │
│     └─────────────────────────────────────────────────────┘     │
│                                                                  │
│     ┌─────────────────────────────────────────────────────┐     │
│     │  Case 3: Same size but different values             │     │
│     │          A(2×2) × B(2×2) ≠ B(2×2) × A(2×2)         │     │
│     └─────────────────────────────────────────────────────┘     │
│                                                                  │
│     Exception: AB = BA when A and B commute (special cases)     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Associative Property Proof Example

For $(AB)C = A(BC)$:

$$A = \begin{bmatrix} 1 & 2 \\ 0 & 1 \end{bmatrix}, B = \begin{bmatrix} 2 & 0 \\ 1 & 3 \end{bmatrix}, C = \begin{bmatrix} 1 & 1 \\ 0 & 2 \end{bmatrix}$$

**Method 1: (AB)C**

$$AB = \begin{bmatrix} 4 & 6 \\ 1 & 3 \end{bmatrix}$$

$$(AB)C = \begin{bmatrix} 4 & 16 \\ 1 & 7 \end{bmatrix}$$

**Method 2: A(BC)**

$$BC = \begin{bmatrix} 2 & 2 \\ 1 & 7 \end{bmatrix}$$

$$A(BC) = \begin{bmatrix} 4 & 16 \\ 1 & 7 \end{bmatrix}$$

**Result**: $(AB)C = A(BC)$ ✓

---

## 4. Distributive Properties

### Left Distribution

$$A(B + C) = AB + AC$$

```
        A(B + C)                    AB + AC
        
    ┌───┐  ┌───┐ ┌───┐          ┌───┐┌───┐   ┌───┐┌───┐
    │ A │× │B+C│ = │ ? │    =    │ A ││ B │ + │ A ││ C │
    └───┘  └───┘ └───┘          └───┘└───┘   └───┘└───┘
```

### Right Distribution

$$(A + B)C = AC + BC$$

```
        (A + B)C                    AC + BC
        
    ┌───┐ ┌───┐   ┌───┐         ┌───┐┌───┐   ┌───┐┌───┐
    │A+B│ × │ C │ = │ ? │   =    │ A ││ C │ + │ B ││ C │
    └───┘ └───┘   └───┘         └───┘└───┘   └───┘└───┘
```

### Example

$$A = \begin{bmatrix} 1 & 2 \end{bmatrix}, B = \begin{bmatrix} 3 \\ 4 \end{bmatrix}, C = \begin{bmatrix} 5 \\ 6 \end{bmatrix}$$

**Verify**: $A(B + C) = AB + AC$

$$B + C = \begin{bmatrix} 8 \\ 10 \end{bmatrix}$$

$$A(B + C) = \begin{bmatrix} 1 & 2 \end{bmatrix}\begin{bmatrix} 8 \\ 10 \end{bmatrix} = [28]$$

$$AB = [11], \quad AC = [17]$$

$$AB + AC = [11] + [17] = [28]$$ ✓

---

## 5. Comparison Table: All Operations

```
┌─────────────────────────────────────────────────────────────────┐
│              COMPARISON OF MATRIX OPERATIONS                     │
├──────────────┬───────────────┬───────────────┬──────────────────┤
│   Property   │   Addition    │    Scalar     │  Multiplication  │
│              │    (A + B)    │     (kA)      │      (AB)        │
├──────────────┼───────────────┼───────────────┼──────────────────┤
│   Closure    │      ✓        │       ✓       │        ✓         │
├──────────────┼───────────────┼───────────────┼──────────────────┤
│ Commutative  │      ✓        │      N/A      │        ✗         │
│              │   A+B = B+A   │               │    AB ≠ BA       │
├──────────────┼───────────────┼───────────────┼──────────────────┤
│ Associative  │      ✓        │       ✓       │        ✓         │
│              │ (A+B)+C       │   (kl)A       │   (AB)C          │
│              │ = A+(B+C)     │   = k(lA)     │   = A(BC)        │
├──────────────┼───────────────┼───────────────┼──────────────────┤
│ Distributive │     N/A       │       ✓       │        ✓         │
│              │               │  k(A+B)=kA+kB │   A(B+C)=AB+AC   │
├──────────────┼───────────────┼───────────────┼──────────────────┤
│   Identity   │      O        │       1       │        I         │
│              │   A + O = A   │   1·A = A     │   AI = IA = A    │
├──────────────┼───────────────┼───────────────┼──────────────────┤
│   Inverse    │     -A        │      N/A      │      A⁻¹         │
│              │  A+(-A) = O   │               │  AA⁻¹ = I        │
└──────────────┴───────────────┴───────────────┴──────────────────┘
```

---

## 6. Special Matrices in Operations

### Identity Matrix Properties

```
┌─────────────────────────────────────────────────────────────────┐
│                  IDENTITY MATRIX (I)                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    AI = A    (right multiplication)                             │
│    IA = A    (left multiplication)                              │
│                                                                  │
│    I² = I                                                        │
│    Iⁿ = I    for any positive integer n                         │
│                                                                  │
│    kI = Scalar Matrix                                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Zero Matrix Properties

```
┌─────────────────────────────────────────────────────────────────┐
│                    ZERO MATRIX (O)                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    A + O = A    (additive identity)                             │
│    A - A = O    (additive inverse result)                       │
│                                                                  │
│    AO = O       (right multiplication)                          │
│    OA = O       (left multiplication)                           │
│                                                                  │
│    kO = O       for any scalar k                                │
│    0·A = O      zero scalar gives zero matrix                   │
│                                                                  │
│    ⚠️ AB = O does NOT imply A = O or B = O                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Powers of Matrices

### Definition

For a square matrix A:

$$A^0 = I$$
$$A^1 = A$$
$$A^2 = A \cdot A$$
$$A^n = \underbrace{A \cdot A \cdot \ldots \cdot A}_{n \text{ times}}$$

### Properties of Powers

| Property | Formula |
|----------|---------|
| Product of powers | $A^m \cdot A^n = A^{m+n}$ |
| Power of power | $(A^m)^n = A^{mn}$ |
| Power of product | $(AB)^n \neq A^n B^n$ (generally) |
| Power of scalar multiple | $(kA)^n = k^n A^n$ |

### Warning: Power of Product

$$(AB)^2 = (AB)(AB) = ABAB \neq A^2B^2$$

Only if AB = BA (matrices commute), then $(AB)^n = A^nB^n$

---

## 8. Cancellation Laws (What DOESN'T Work)

### Cannot Cancel in General

```
┌─────────────────────────────────────────────────────────────────┐
│              ⚠️ CANCELLATION DOES NOT WORK ⚠️                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    From AB = AC, we CANNOT conclude B = C                       │
│                                                                  │
│    Example:                                                      │
│                                                                  │
│    A = [1  0]    B = [1  2]    C = [1  2]                      │
│        [0  0]        [3  4]        [5  6]                      │
│                                                                  │
│    AB = [1  2]   AC = [1  2]                                   │
│         [0  0]        [0  0]                                   │
│                                                                  │
│    AB = AC, but B ≠ C !                                        │
│                                                                  │
│    Reason: A is singular (non-invertible)                       │
│                                                                  │
│    ✓ Cancellation works only if A is invertible                │
│      If A⁻¹ exists: AB = AC → A⁻¹AB = A⁻¹AC → B = C           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Summary: Algebraic Structure

### Matrices Form a Ring (Almost)

```
┌─────────────────────────────────────────────────────────────────┐
│                    ALGEBRAIC STRUCTURE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    The set of n×n matrices with real entries forms:            │
│                                                                  │
│    ┌───────────────────────────────────────────────────────┐    │
│    │  A NON-COMMUTATIVE RING WITH IDENTITY                 │    │
│    └───────────────────────────────────────────────────────┘    │
│                                                                  │
│    Properties:                                                   │
│    • Closed under addition and multiplication                   │
│    • Addition is commutative and associative                    │
│    • Additive identity (O) and inverses (-A) exist             │
│    • Multiplication is associative                              │
│    • Multiplication is NOT commutative                          │
│    • Multiplicative identity (I) exists                         │
│    • Distributive laws hold                                     │
│                                                                  │
│    NOT a field because:                                          │
│    • Not all non-zero matrices have multiplicative inverses    │
│    • Multiplication is not commutative                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Operation | Properties That Hold | Properties That DON'T Hold |
|-----------|---------------------|----------------------------|
| Addition | Closure, Commutative, Associative, Identity (O), Inverse (-A) | N/A |
| Scalar Mult. | Closure, Associative, Distributive, Identity (1) | N/A |
| Multiplication | Closure, Associative, Distributive, Identity (I) | Commutative, Cancellation |

### Key Formulas

| Formula | Valid? |
|---------|--------|
| $A + B = B + A$ | ✓ Always |
| $AB = BA$ | ✗ Generally not |
| $(AB)C = A(BC)$ | ✓ Always |
| $A(B+C) = AB + AC$ | ✓ Always |
| $(A+B)^2 = A^2 + 2AB + B^2$ | ✗ Only if AB = BA |
| $(AB)^{-1} = B^{-1}A^{-1}$ | ✓ If inverses exist |

---

## ❓ Quick Revision Questions

1. **Is matrix addition commutative? Is matrix multiplication commutative?**
   <details>
   <summary>Click for Answer</summary>
   Matrix addition IS commutative (A + B = B + A).
   Matrix multiplication is NOT commutative (AB ≠ BA in general).
   </details>

2. **What is the additive identity for matrices? What is the multiplicative identity?**
   <details>
   <summary>Click for Answer</summary>
   Additive identity: Zero matrix O (A + O = A)
   Multiplicative identity: Identity matrix I (AI = IA = A)
   </details>

3. **If AB = AC, can we conclude B = C?**
   <details>
   <summary>Click for Answer</summary>
   No! Cancellation doesn't work in general for matrices. We can only conclude B = C if A is invertible (has an inverse).
   </details>

4. **Simplify: 2A + 3A - A**
   <details>
   <summary>Click for Answer</summary>
   Using distributive property: (2 + 3 - 1)A = 4A
   </details>

5. **Is $(A + B)^2 = A^2 + 2AB + B^2$ true for matrices?**
   <details>
   <summary>Click for Answer</summary>
   No! $(A+B)^2 = A^2 + AB + BA + B^2$. This equals $A^2 + 2AB + B^2$ only if AB = BA (matrices commute).
   </details>

6. **If A² = O, is A = O?**
   <details>
   <summary>Click for Answer</summary>
   Not necessarily! For example, $A = \begin{bmatrix} 0 & 1 \\ 0 & 0 \end{bmatrix}$ gives $A^2 = O$ but $A \neq O$. Such matrices are called nilpotent.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Matrix Multiplication](03-matrix-multiplication.md) | [Unit 2: Operations](./README.md) | [Transpose of Matrix →](05-transpose-of-matrix.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
