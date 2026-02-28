# Chapter 3.3: LU Decomposition

[← Previous: Gauss-Jordan](02-gauss-jordan.md) | [Next: Jacobi Method →](04-jacobi-method.md)

---

## Overview

**LU Decomposition** factors a matrix $A$ into the product of a **lower triangular** matrix $L$ and an **upper triangular** matrix $U$: $A = LU$. This factorization is computed once; solving $Ax = b$ then reduces to two cheap triangular solves. It is highly efficient when solving multiple systems with the same coefficient matrix.

---

## 1. The Factorization

$$A = LU$$

```
  ┌             ┐     ┌             ┐   ┌             ┐
  │ a₁₁ a₁₂ a₁₃│     │ 1   0   0  │   │ u₁₁ u₁₂ u₁₃│
  │ a₂₁ a₂₂ a₂₃│  =  │ l₂₁ 1   0  │ × │  0  u₂₂ u₂₃│
  │ a₃₁ a₃₂ a₃₃│     │ l₃₁ l₃₂ 1  │   │  0   0  u₃₃│
  └             ┘     └             ┘   └             ┘
        A                   L                  U
                      (lower triangular    (upper triangular)
                       with 1s on diagonal)
```

**Doolittle**: $L$ has 1s on diagonal (shown above)
**Crout**: $U$ has 1s on diagonal

---

## 2. Solving $Ax = b$ Using LU

Since $A = LU$, the system $Ax = b$ becomes $LUx = b$.

**Step 1**: Solve $Ly = b$ (forward substitution) — $O(n^2/2)$
**Step 2**: Solve $Ux = y$ (back substitution) — $O(n^2/2)$

```
  Ax = b
   │
   ▼
  LUx = b
   │
   ├──► Step 1: Ly = b  (forward sub)  →  find y
   │
   └──► Step 2: Ux = y  (back sub)     →  find x
```

**Key advantage**: If we need to solve $Ax = b_1$, $Ax = b_2$, ..., $Ax = b_k$, the LU factorization is done **once** ($O(n^3/3)$), and each subsequent solve is only $O(n^2)$.

---

## 3. Computing LU (Doolittle's Method)

The elements of $L$ and $U$ are found by matching $A = LU$ element by element.

For $U$ (row $i$, column $j$ where $j \geq i$):

$$u_{ij} = a_{ij} - \sum_{k=1}^{i-1} l_{ik} u_{kj}$$

For $L$ (row $i$, column $j$ where $i > j$):

$$l_{ij} = \frac{1}{u_{jj}} \left(a_{ij} - \sum_{k=1}^{j-1} l_{ik} u_{kj}\right)$$

---

## 4. Worked Example

Factor $A = \begin{bmatrix} 2 & 1 & 1 \\ 4 & 3 & 3 \\ 8 & 7 & 9 \end{bmatrix}$ and solve $Ax = \begin{bmatrix} 4 \\ 10 \\ 24 \end{bmatrix}$

### Step 1: Compute U (first row) and L (first column)

$U$ first row: $u_{11} = 2, u_{12} = 1, u_{13} = 1$

$L$ first column: $l_{21} = 4/2 = 2, \; l_{31} = 8/2 = 4$

### Step 2: Complete U and L

$u_{22} = a_{22} - l_{21}u_{12} = 3 - 2(1) = 1$
$u_{23} = a_{23} - l_{21}u_{13} = 3 - 2(1) = 1$
$l_{32} = (a_{32} - l_{31}u_{12})/u_{22} = (7 - 4(1))/1 = 3$
$u_{33} = a_{33} - l_{31}u_{13} - l_{32}u_{23} = 9 - 4(1) - 3(1) = 2$

### Result

$$L = \begin{bmatrix} 1 & 0 & 0 \\ 2 & 1 & 0 \\ 4 & 3 & 1 \end{bmatrix}, \quad U = \begin{bmatrix} 2 & 1 & 1 \\ 0 & 1 & 1 \\ 0 & 0 & 2 \end{bmatrix}$$

### Step 3: Forward substitution $Ly = b$

$$y_1 = 4$$
$$y_2 = 10 - 2(4) = 2$$
$$y_3 = 24 - 4(4) - 3(2) = 2$$

### Step 4: Back substitution $Ux = y$

$$x_3 = 2/2 = 1$$
$$x_2 = (2 - 1)/1 = 1$$
$$x_1 = (4 - 1 - 1)/2 = 1$$

**Solution**: $x = [1, 1, 1]^T$

---

## 5. LU with Pivoting (PA = LU)

When pivoting is needed: $PA = LU$ where $P$ is a permutation matrix.

```
  Without pivoting:  A = LU       (may fail if pivot = 0)
  With pivoting:     PA = LU      (always works for nonsingular A)

  P records the row swaps performed during elimination.
```

---

## 6. Cholesky Decomposition (Symmetric Positive Definite)

If $A$ is **symmetric positive definite** ($A = A^T$, all eigenvalues > 0):

$$A = LL^T$$

where $L$ is lower triangular. This is **twice as fast** and needs **half the storage** of general LU.

**Formulas**:

$$l_{jj} = \sqrt{a_{jj} - \sum_{k=1}^{j-1} l_{jk}^2}$$

$$l_{ij} = \frac{1}{l_{jj}} \left(a_{ij} - \sum_{k=1}^{j-1} l_{ik} l_{jk}\right), \quad i > j$$

---

## 7. Comparison of Direct Methods

| Method | Factorization Cost | Solve Cost (per RHS) | Best For |
|--------|:-----------------:|:-------------------:|----------|
| Gaussian Elimination | $n^3/3$ | $n^2$ | Single system |
| Gauss-Jordan | $n^3/2$ | — | Matrix inverse |
| LU Decomposition | $n^3/3$ | $n^2$ | Multiple RHS |
| Cholesky ($LL^T$) | $n^3/6$ | $n^2$ | Symmetric positive definite |

---

## 8. Real-World Applications

| Application | Why LU |
|-------------|--------|
| FEA/Structural analysis | Same stiffness matrix, many load vectors |
| Circuit simulation | Same network matrix, varying inputs |
| Control systems | Solve $Ax = b$ repeatedly in real-time |
| Computing determinant | $\det(A) = \prod u_{ii}$ |
| Computing inverse | Solve $AX = I$ column by column |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Factorization** | $A = LU$ (Doolittle: $L$ has 1s on diagonal) |
| **Cost** | $O(n^3/3)$ for factorization, $O(n^2)$ per solve |
| **Advantage** | Efficient for multiple right-hand sides |
| **With pivoting** | $PA = LU$ |
| **Symmetric PD** | Cholesky: $A = LL^T$, cost $n^3/6$ |
| **Determinant** | $\det(A) = \prod_{i} u_{ii}$ |

---

## Quick Revision Questions

1. **Explain the advantage of LU decomposition over Gaussian elimination when solving $Ax = b$ for multiple right-hand sides.**

2. **Find the LU decomposition of $A = \begin{bmatrix} 1 & 2 \\ 3 & 5 \end{bmatrix}$.**

3. **What is Cholesky decomposition? When can it be used?**

4. **Given $L$ and $U$, how do you compute $\det(A)$?**

5. **Why is $PA = LU$ (with pivoting) more numerically stable than $A = LU$?**

6. **Compare the operation counts of LU, Cholesky, and Gaussian elimination.**

---

[← Previous: Gauss-Jordan](02-gauss-jordan.md) | [Next: Jacobi Method →](04-jacobi-method.md)
