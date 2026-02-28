# 7.3 Least Squares Problems

[← Previous: Pseudo-Inverse](02-pseudo-inverse.md) · [Back to README](../README.md) · [Next: Data Compression & Low-Rank Approximation →](04-data-compression.md)

---

## Chapter Overview

When $A\mathbf{x} = \mathbf{b}$ has **no exact solution** (overdetermined system), we seek the vector $\hat{\mathbf{x}}$ that minimises $\|A\mathbf{x} - \mathbf{b}\|^2$. This is the **least squares** problem — fundamental in data fitting, regression, and scientific computing.

---

## 1. The Problem

Given $A$ ($m \times n$, $m > n$) and $\mathbf{b} \in \mathbb{R}^m$:

$$\min_{\mathbf{x} \in \mathbb{R}^n} \|A\mathbf{x} - \mathbf{b}\|^2$$

The system $A\mathbf{x} = \mathbf{b}$ typically has no solution when there are more equations than unknowns.

```
  Overdetermined system: more equations than unknowns
  
  3x + 2y = 5      ┐
  1x - 1y = 2      ├── 3 equations, 2 unknowns
  2x + 1y = 4      ┘   (generally no exact solution)
  
  Best we can do: minimise the total squared error
  
  E = (3x+2y-5)² + (x-y-2)² + (2x+y-4)²
```

---

## 2. Geometric Solution: Projection

$\hat{\mathbf{x}}$ minimises $\|A\mathbf{x} - \mathbf{b}\|$ iff $A\hat{\mathbf{x}}$ is the **orthogonal projection** of $\mathbf{b}$ onto $\text{Col}(A)$.

The residual $\mathbf{r} = \mathbf{b} - A\hat{\mathbf{x}}$ must be orthogonal to $\text{Col}(A)$:

$$A^T(\mathbf{b} - A\hat{\mathbf{x}}) = \mathbf{0}$$

```
  Geometry of least squares:
  
           b              b is in ℝᵐ
          /|              Col(A) is a subspace of ℝᵐ
         / |
        /  | r = b - Aẑ   (residual, perpendicular
       /   |               to Col(A))
      /    |
     /     ▼
  ──┼──────·──────── Col(A)
     \   Aẑ = proj
      \  (closest point in Col(A) to b)
```

---

## 3. The Normal Equations

> **Theorem.** The least squares solution satisfies:
>
> $$\boxed{A^T A \hat{\mathbf{x}} = A^T \mathbf{b}}$$
>
> These are the **normal equations**.

If $A$ has **full column rank** (rank $n$), then $A^TA$ is invertible and:

$$\hat{\mathbf{x}} = (A^TA)^{-1} A^T \mathbf{b}$$

---

## 4. Worked Example: Line Fitting

Fit a line $y = c_0 + c_1 x$ through points $(1, 1), (2, 2), (3, 4)$.

$$\begin{pmatrix} 1 & 1 \\ 1 & 2 \\ 1 & 3 \end{pmatrix}\begin{pmatrix} c_0 \\ c_1 \end{pmatrix} = \begin{pmatrix} 1 \\ 2 \\ 4 \end{pmatrix}$$

**Normal equations:** $A^TA\hat{\mathbf{x}} = A^T\mathbf{b}$

$$A^TA = \begin{pmatrix} 3 & 6 \\ 6 & 14 \end{pmatrix}, \quad A^T\mathbf{b} = \begin{pmatrix} 7 \\ 17 \end{pmatrix}$$

$$\begin{pmatrix} 3 & 6 \\ 6 & 14 \end{pmatrix}\begin{pmatrix} c_0 \\ c_1 \end{pmatrix} = \begin{pmatrix} 7 \\ 17 \end{pmatrix}$$

$$\hat{c}_0 = -1/3, \quad \hat{c}_1 = 3/2$$

Best fit line: $y = -\frac{1}{3} + \frac{3}{2}x$

```
  y
  4│         *
  3│       /
  2│    */
  1│  */
  0│/
   └──────────→ x
   0  1  2  3
   
  * = data points
  / = best fit line y = -1/3 + 3/2 x
```

---

## 5. Least Squares via SVD

If $A = U\Sigma V^T$, then:

$$\hat{\mathbf{x}} = A^+\mathbf{b} = V\Sigma^+ U^T \mathbf{b}$$

**Advantages over normal equations:**
- Numerically more stable
- Works even when $A^TA$ is nearly singular
- Does not form $A^TA$ (which can lose accuracy by squaring condition number)

---

## 6. Least Squares via QR Factorisation

Write $A = QR$ (thin QR, $Q$ is $m \times n$ with orthonormal columns):

$$A^TA\hat{\mathbf{x}} = A^T\mathbf{b} \implies R^TQ^TQR\hat{\mathbf{x}} = R^TQ^T\mathbf{b} \implies R\hat{\mathbf{x}} = Q^T\mathbf{b}$$

Solve by back substitution — fast and stable!

---

## 7. General Least Squares: Polynomial Fitting

Fit $y = c_0 + c_1 x + c_2 x^2$ to data. The design matrix becomes:

$$A = \begin{pmatrix} 1 & x_1 & x_1^2 \\ 1 & x_2 & x_2^2 \\ \vdots & \vdots & \vdots \\ 1 & x_m & x_m^2 \end{pmatrix}$$

This is a **Vandermonde matrix**. Same normal equation approach.

---

## 8. Residual and Goodness of Fit

| Quantity | Formula | Meaning |
|----------|---------|---------|
| Residual vector | $\mathbf{r} = \mathbf{b} - A\hat{\mathbf{x}}$ | Error at each data point |
| Residual norm | $\|\mathbf{r}\| = \|\mathbf{b} - A\hat{\mathbf{x}}\|$ | Total error magnitude |
| $R^2$ (coeff. of determination) | $1 - \frac{\|\mathbf{r}\|^2}{\|\mathbf{b} - \bar{b}\|^2}$ | Fraction of variance explained |
| Projection matrix | $P = A(A^TA)^{-1}A^T$ | Projects $\mathbf{b}$ onto Col$(A)$ |

---

## 9. Three Methods Compared

| Method | Formula | Best for |
|--------|---------|----------|
| Normal equations | $(A^TA)^{-1}A^T\mathbf{b}$ | Small, well-conditioned problems |
| QR factorisation | $R\hat{\mathbf{x}} = Q^T\mathbf{b}$ | General use, good stability |
| SVD | $V\Sigma^+U^T\mathbf{b}$ | Ill-conditioned or rank-deficient |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Least squares problem | Minimise $\|A\mathbf{x} - \mathbf{b}\|^2$ |
| Geometric view | Project $\mathbf{b}$ onto Col$(A)$ |
| Normal equations | $A^TA\hat{\mathbf{x}} = A^T\mathbf{b}$ |
| Full rank solution | $\hat{\mathbf{x}} = (A^TA)^{-1}A^T\mathbf{b}$ |
| SVD approach | $\hat{\mathbf{x}} = A^+\mathbf{b}$ |
| QR approach | $R\hat{\mathbf{x}} = Q^T\mathbf{b}$ |
| Best fit line | Vandermonde system with $A$ = $(1, x_i)$ columns |
| Residual | $\mathbf{r} \perp \text{Col}(A)$ |

---

## Quick Revision Questions

1. Derive the normal equations from the condition $\mathbf{r} \perp \text{Col}(A)$.

2. Fit a line $y = a + bx$ to points $(0, 1), (1, 3), (2, 4), (3, 4)$.

3. Why is the SVD method more numerically stable than normal equations?

4. Show that the projection matrix $P = A(A^TA)^{-1}A^T$ satisfies $P^2 = P$.

5. If $A$ has orthonormal columns, what does the least squares solution simplify to?

---

[← Previous: Pseudo-Inverse](02-pseudo-inverse.md) · [Back to README](../README.md) · [Next: Data Compression & Low-Rank Approximation →](04-data-compression.md)
