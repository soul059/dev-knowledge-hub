# Chapter 8.2: Jacobi's Eigenvalue Method

[← Previous: Power Method](01-power-method.md) | [Next: QR Algorithm →](03-qr-algorithm.md)

---

## Overview

**Jacobi's eigenvalue method** finds ALL eigenvalues and eigenvectors of a **real symmetric matrix** by applying a sequence of orthogonal rotations (Givens rotations) to systematically zero out off-diagonal elements. The diagonal converges to the eigenvalues.

---

## 1. Key Idea

For a symmetric matrix $A$:

$$A^{(k+1)} = R_k^T A^{(k)} R_k$$

where $R_k$ is a **Givens rotation** chosen to zero out the largest off-diagonal element.

Since $R_k$ is orthogonal, eigenvalues are preserved. Eventually:

$$A^{(k)} \to \text{diag}(\lambda_1, \lambda_2, \ldots, \lambda_n)$$

---

## 2. Givens Rotation Matrix

To zero out element $a_{pq}$ (with $p \neq q$), use:

$$R = I + (\cos\theta - 1)(e_p e_p^T + e_q e_q^T) + \sin\theta(e_p e_q^T - e_q e_p^T)$$

The rotation angle $\theta$ is chosen so that $a'_{pq} = 0$:

$$\boxed{\tan(2\theta) = \frac{2a_{pq}}{a_{pp} - a_{qq}}}$$

If $a_{pp} = a_{qq}$: $\theta = \pm \pi/4$.

---

## 3. Algorithm

```
  Input: Symmetric matrix A (n × n)

  Repeat:
    1. Find largest off-diagonal |a_pq|
    2. Compute θ from tan(2θ) = 2a_pq/(a_pp - a_qq)
    3. Apply rotation: A' = R^T A R
       (only rows/columns p,q change)
    4. Until all off-diagonal elements < tolerance

  Output: Diagonal = eigenvalues
          Product of all rotations = eigenvectors
```

---

## 4. Worked Example

Find eigenvalues of $A = \begin{bmatrix} 4 & 2 \\ 2 & 1 \end{bmatrix}$.

**Step 1**: Largest off-diagonal: $a_{12} = 2$.

**Step 2**: $\tan(2\theta) = \frac{2(2)}{4-1} = \frac{4}{3}$

$2\theta = \arctan(4/3) = 53.13°$, so $\theta = 26.57°$

$\cos\theta = 0.8944, \quad \sin\theta = 0.4472$

**Step 3**: Apply rotation:

$$R = \begin{bmatrix} 0.8944 & 0.4472 \\ -0.4472 & 0.8944 \end{bmatrix}$$

$$A' = R^T A R = \begin{bmatrix} 5 & 0 \\ 0 & 0 \end{bmatrix}$$

**Eigenvalues**: $\lambda_1 = 5, \lambda_2 = 0$ ✓

**Check**: $\det(A) = 4(1) - 2(2) = 0$ and $\text{tr}(A) = 5$. Confirms $\lambda = 5, 0$.

---

## 5. Convergence

- **Quadratic convergence** (eventually): the sum of squares of off-diagonal elements decreases by a guaranteed factor per sweep
- A **sweep** = zeroing all off-diagonal elements once (each one may reintroduce others)
- Total cost: $O(n^3)$ per sweep, typically $5$–$10$ sweeps needed
- **Only for symmetric matrices** (guaranteed real eigenvalues)

---

## 6. Properties

| Property | Detail |
|----------|--------|
| Finds | ALL eigenvalues AND eigenvectors |
| Works for | Real symmetric matrices only |
| Preserves | Eigenvalues (similarity transform) |
| Convergence | Quadratic (eventually) |
| Cost | $O(n^3)$ per sweep |
| Advantage | Numerically very stable |
| Disadvantage | Slower than QR for large $n$ |

---

## 7. The Threshold Jacobi Method

Instead of always zeroing the **largest** off-diagonal (expensive to search), use a **threshold**: zero any element $|a_{pq}| > \tau$, decreasing $\tau$ over sweeps. This is faster in practice.

---

## Summary Table

| Property | Value |
|----------|-------|
| **Method** | Orthogonal rotations to diagonalise |
| **Input** | Real symmetric matrix |
| **Output** | All eigenvalues and eigenvectors |
| **Rotation angle** | $\tan(2\theta) = 2a_{pq}/(a_{pp} - a_{qq})$ |
| **Convergence** | Quadratic; $5$–$10$ sweeps typical |
| **Stability** | Excellent |

---

## Quick Revision Questions

1. **Describe the Jacobi eigenvalue algorithm step by step.**

2. **Find all eigenvalues of $\begin{bmatrix} 3 & 1 \\ 1 & 3 \end{bmatrix}$ using one Jacobi rotation.**

3. **Why does Jacobi's method only work for symmetric matrices?**

4. **Derive the rotation angle formula $\tan(2\theta) = 2a_{pq}/(a_{pp} - a_{qq})$.**

5. **How are eigenvectors obtained from the Jacobi method?**

6. **Compare Jacobi's method with the power method for finding eigenvalues.**

---

[← Previous: Power Method](01-power-method.md) | [Next: QR Algorithm →](03-qr-algorithm.md)
