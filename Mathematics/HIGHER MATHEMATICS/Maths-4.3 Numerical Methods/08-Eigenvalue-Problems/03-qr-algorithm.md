# Chapter 8.3: QR Algorithm

[← Previous: Jacobi's Method](02-jacobi-method.md) | [Next: Unit 9 — Least Squares →](../09-Curve-Fitting/01-least-squares.md)

---

## Overview

The **QR algorithm** is the most important eigenvalue algorithm in modern numerical linear algebra. It finds ALL eigenvalues of a general (not necessarily symmetric) matrix by repeatedly factoring $A = QR$ and forming $A' = RQ$.

---

## 1. QR Decomposition

Any square matrix $A$ can be factored as:

$$A = QR$$

where:
- $Q$ is **orthogonal** ($Q^T Q = I$)
- $R$ is **upper triangular**

Methods to compute: Gram-Schmidt, Householder reflections, Givens rotations.

---

## 2. Basic QR Algorithm

```
  Input: Matrix A
  
  Repeat:
    1. Factor: A_k = Q_k R_k    (QR decomposition)
    2. Reverse: A_{k+1} = R_k Q_k  (multiply in reverse order)
  
  Until A_k converges to upper triangular form
  
  Output: Diagonal of A_k = eigenvalues
```

**Why it works**: $A_{k+1} = R_k Q_k = Q_k^T (Q_k R_k) Q_k = Q_k^T A_k Q_k$ — this is a **similarity transformation**, preserving eigenvalues!

---

## 3. Convergence

Under typical conditions, $A_k$ converges to:

- **Upper triangular** (Schur form) for general matrices
- **Diagonal** for symmetric matrices

The **sub-diagonal** elements converge to zero at the rate:

$$|a_{i+1,i}^{(k)}| \sim \left|\frac{\lambda_{i+1}}{\lambda_i}\right|^k$$

---

## 4. Shifts — The QR Algorithm with Shifts

To accelerate convergence, apply a **shift**:

$$A_k - \sigma_k I = Q_k R_k, \quad A_{k+1} = R_k Q_k + \sigma_k I$$

Common shift strategies:
- **Rayleigh quotient shift**: $\sigma_k = a_{nn}^{(k)}$ (last diagonal element)
- **Wilkinson shift**: uses the eigenvalue of the bottom $2 \times 2$ block closest to $a_{nn}$

With shifts: **cubic convergence** for symmetric matrices!

---

## 5. Practical Implementation

For efficiency, first reduce $A$ to **Hessenberg form** (upper triangular + one sub-diagonal):

```
  General:          Hessenberg:       Upper triangular:
  ╔═══════════╗    ╔═══════════╗     ╔═══════════╗
  ║ * * * * * ║    ║ * * * * * ║     ║ * * * * * ║
  ║ * * * * * ║    ║ * * * * * ║     ║ 0 * * * * ║
  ║ * * * * * ║ →  ║ 0 * * * * ║  →  ║ 0 0 * * * ║
  ║ * * * * * ║    ║ 0 0 * * * ║     ║ 0 0 0 * * ║
  ║ * * * * * ║    ║ 0 0 0 * * ║     ║ 0 0 0 0 * ║
  ╚═══════════╝    ╚═══════════╝     ╚═══════════╝
    O(n³)             O(n²) per        Eigenvalues
    one-time          QR step          on diagonal
```

- Hessenberg reduction: $O(n^3)$ (done once)
- Each QR iteration on Hessenberg: $O(n^2)$ (not $O(n^3)$!)
- Total: $O(n^3)$ for all eigenvalues

---

## 6. Worked Example (2×2)

Find eigenvalues of $A = \begin{bmatrix} 5 & 1 \\ 4 & 2 \end{bmatrix}$.

**Iteration 1**: QR factorisation:

$A = QR$ where $Q = \frac{1}{\sqrt{41}}\begin{bmatrix} 5 & 4 \\ 4 & -5 \end{bmatrix}$, $R = \frac{1}{\sqrt{41}}\begin{bmatrix} 41 & 13 \\ 0 & -1 \end{bmatrix}$

Wait — let me use correct QR. Using Givens rotation to zero $a_{21} = 4$:

$r = \sqrt{5^2 + 4^2} = \sqrt{41}$, $c = 5/\sqrt{41}$, $s = 4/\sqrt{41}$

$$R = \begin{bmatrix} \sqrt{41} & 13/\sqrt{41} \\ 0 & 1/\sqrt{41} \end{bmatrix}$$

Then $A_1 = RQ$:

$$A_1 = RQ = \begin{bmatrix} 5.683 & 0.317 \\ 0.976 & 1.317 \end{bmatrix}$$

After a few more iterations:

$$A_k \to \begin{bmatrix} 6 & * \\ \to 0 & 1 \end{bmatrix}$$

**Eigenvalues**: $\lambda_1 = 6, \lambda_2 = 1$ (verify: trace = 7, det = 6 ✓)

---

## 7. Comparison of Eigenvalue Methods

| Method | Finds | Matrix type | Cost | Convergence |
|--------|-------|------------|------|-------------|
| **Power** | 1 eigenvalue | Any | $O(n^2)/\text{iter}$ | Linear |
| **Jacobi** | All | Symmetric | $O(n^3)/\text{sweep}$ | Quadratic |
| **QR** | All | **Any** | $O(n^3)$ total | Cubic (shifted) |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Finds** | ALL eigenvalues |
| **Works for** | Any square matrix (general or symmetric) |
| **Iteration** | $A = QR$, then $A' = RQ$ |
| **Convergence** | Linear (basic), cubic (shifted) |
| **Pre-processing** | Hessenberg reduction ($O(n^3)$, done once) |
| **Total cost** | $O(n^3)$ |
| **Modern standard** | THE algorithm for eigenvalue computation |

---

## Quick Revision Questions

1. **What is the QR decomposition and how is it used in the QR algorithm?**

2. **Show that $A_{k+1} = R_k Q_k$ is similar to $A_k$.**

3. **What is a Hessenberg matrix and why is pre-reduction to Hessenberg form important?**

4. **Explain the Wilkinson shift and its effect on convergence.**

5. **Compare the QR algorithm with the power method and Jacobi's method.**

6. **Apply one step of the unshifted QR algorithm to $\begin{bmatrix} 3 & 1 \\ 1 & 3 \end{bmatrix}$.**

---

[← Previous: Jacobi's Method](02-jacobi-method.md) | [Next: Unit 9 — Least Squares →](../09-Curve-Fitting/01-least-squares.md)
