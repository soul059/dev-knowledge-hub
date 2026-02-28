# Chapter 8.1: Power Method

[← Previous: Boundary Value Problems](../07-Ordinary-DEs/06-boundary-value-problems.md) | [Next: Jacobi's Eigenvalue Method →](02-jacobi-method.md)

---

## Overview

The **Power Method** finds the **dominant eigenvalue** (largest in absolute value) and its corresponding eigenvector of a matrix through repeated matrix-vector multiplication. It is the simplest iterative eigenvalue algorithm and the foundation of Google's PageRank.

---

## 1. The Algorithm

Given matrix $A$ ($n \times n$):

1. Start with an initial vector $\mathbf{x}^{(0)}$ (non-zero, often $[1, 1, \ldots, 1]^T$)
2. Iterate: $\mathbf{y}^{(k)} = A \mathbf{x}^{(k-1)}$
3. Normalise: $\mathbf{x}^{(k)} = \frac{\mathbf{y}^{(k)}}{\|\mathbf{y}^{(k)}\|_\infty}$ (divide by largest component)
4. The normalisation factor converges to $\lambda_1$ (dominant eigenvalue)

$$\boxed{\lambda_1 = \lim_{k \to \infty} \frac{\|\mathbf{y}^{(k)}\|_\infty}{\|\mathbf{x}^{(k-1)}\|_\infty}}$$

---

## 2. Worked Example

Find the dominant eigenvalue of $A = \begin{bmatrix} 2 & 1 \\ 1 & 3 \end{bmatrix}$.

Start: $\mathbf{x}^{(0)} = \begin{bmatrix} 1 \\ 1 \end{bmatrix}$

**Iteration 1:**
$$\mathbf{y}^{(1)} = \begin{bmatrix} 2 & 1 \\ 1 & 3 \end{bmatrix}\begin{bmatrix} 1 \\ 1 \end{bmatrix} = \begin{bmatrix} 3 \\ 4 \end{bmatrix}$$

$\lambda \approx 4$, $\quad \mathbf{x}^{(1)} = \begin{bmatrix} 0.75 \\ 1 \end{bmatrix}$

**Iteration 2:**
$$\mathbf{y}^{(2)} = \begin{bmatrix} 2 & 1 \\ 1 & 3 \end{bmatrix}\begin{bmatrix} 0.75 \\ 1 \end{bmatrix} = \begin{bmatrix} 2.5 \\ 3.75 \end{bmatrix}$$

$\lambda \approx 3.75$, $\quad \mathbf{x}^{(2)} = \begin{bmatrix} 0.667 \\ 1 \end{bmatrix}$

**Iteration 3:**
$$\mathbf{y}^{(3)} = A\begin{bmatrix} 0.667 \\ 1 \end{bmatrix} = \begin{bmatrix} 2.334 \\ 3.667 \end{bmatrix}$$

$\lambda \approx 3.667$, $\quad \mathbf{x}^{(3)} = \begin{bmatrix} 0.636 \\ 1 \end{bmatrix}$

Continuing: $\lambda \to 3.618$ (exact: $\frac{5+\sqrt{5}}{2} = 3.618$)

Eigenvector: $\begin{bmatrix} 0.618 \\ 1 \end{bmatrix}$ (ratio converges to $(\sqrt{5}-1)/2$)

---

## 3. Convergence Rate

The convergence rate is:

$$\text{error at step } k \sim \left|\frac{\lambda_2}{\lambda_1}\right|^k$$

- **Fast** if $|\lambda_2/\lambda_1| \ll 1$ (dominant eigenvalue is well-separated)
- **Slow** if $|\lambda_2| \approx |\lambda_1|$

---

## 4. Inverse Power Method

To find the **smallest** eigenvalue, apply the power method to $A^{-1}$:

$$A \mathbf{y}^{(k)} = \mathbf{x}^{(k-1)} \quad \text{(solve instead of multiply)}$$

The dominant eigenvalue of $A^{-1}$ is $1/\lambda_{\min}(A)$.

### Shifted Inverse Power Method

To find the eigenvalue **closest to a shift $\sigma$**:

Apply inverse power method to $(A - \sigma I)^{-1}$. This converges to the eigenvalue nearest to $\sigma$.

---

## 5. Deflation

After finding $\lambda_1$ and $\mathbf{v}_1$, find the next eigenvalue by **deflation**:

$$A_2 = A - \lambda_1 \frac{\mathbf{v}_1 \mathbf{v}_1^T}{\mathbf{v}_1^T \mathbf{v}_1}$$

Apply the power method to $A_2$ to get $\lambda_2$.

---

## 6. Limitations

```
  ┌─────────────────────────────────────────┐
  │       When Power Method Fails           │
  ├─────────────────────────────────────────┤
  │ • Complex dominant eigenvalue           │
  │ • Two eigenvalues with same |λ|        │
  │ • Initial guess orthogonal to v₁       │
  │ • Very slow if |λ₂/λ₁| ≈ 1            │
  └─────────────────────────────────────────┘
```

---

## Summary Table

| Property | Value |
|----------|-------|
| **Finds** | Dominant eigenvalue $\lambda_1$ |
| **Convergence** | $\sim |\lambda_2/\lambda_1|^k$ |
| **Inverse version** | Finds smallest eigenvalue |
| **Shifted inverse** | Finds eigenvalue nearest to $\sigma$ |
| **Cost per iteration** | $O(n^2)$ (matrix-vector multiply) |
| **Limitation** | Only finds one eigenvalue at a time |

---

## Quick Revision Questions

1. **Describe the power method algorithm step by step.**

2. **Find the dominant eigenvalue of $\begin{bmatrix} 4 & 1 \\ 2 & 3 \end{bmatrix}$ using 3 iterations of the power method.**

3. **How does the ratio $|\lambda_2/\lambda_1|$ affect convergence speed?**

4. **What is the inverse power method and when is it used?**

5. **Explain the shifted inverse power method.**

6. **Why might the power method fail? List three scenarios.**

---

[← Previous: Boundary Value Problems](../07-Ordinary-DEs/06-boundary-value-problems.md) | [Next: Jacobi's Eigenvalue Method →](02-jacobi-method.md)
