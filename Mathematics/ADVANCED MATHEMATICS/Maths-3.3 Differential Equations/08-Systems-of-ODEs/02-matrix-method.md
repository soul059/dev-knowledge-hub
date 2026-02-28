# Chapter 8.2: Matrix Method (Eigenvalue Method)

[← Previous: Introduction & First-Order Systems](01-introduction-and-first-order-systems.md) | [Back to README](../README.md) | [Next: Phase Plane Analysis →](03-phase-plane.md)

---

## Overview

The **eigenvalue method** is the most systematic approach for solving constant-coefficient linear systems $\mathbf{x}' = A\mathbf{x}$. We find eigenvalues and eigenvectors of the coefficient matrix $A$ and construct solutions of the form $\mathbf{x} = \mathbf{v}e^{\lambda t}$.

---

## 1. The Eigenvalue Approach

For $\mathbf{x}' = A\mathbf{x}$, try $\mathbf{x} = \mathbf{v}e^{\lambda t}$:

$$\lambda \mathbf{v} e^{\lambda t} = A\mathbf{v} e^{\lambda t}$$
$$A\mathbf{v} = \lambda \mathbf{v}$$

This is the standard eigenvalue problem:

$$\det(A - \lambda I) = 0 \quad \text{(characteristic equation)}$$

```
  EIGENVALUE METHOD WORKFLOW

  ┌─────────────────────────┐
  │ Write system x' = Ax    │
  └──────────┬──────────────┘
             ▼
  ┌─────────────────────────┐
  │ Find det(A - λI) = 0    │
  │ → eigenvalues λ₁, λ₂   │
  └──────────┬──────────────┘
             ▼
  ┌─────────────────────────────────┐
  │ For each λᵢ, solve              │
  │ (A - λᵢI)v = 0 → eigenvector   │
  └──────────┬──────────────────────┘
             ▼
  ┌────────────────────────────────────┐
  │ General solution:                   │
  │ x = c₁v₁e^(λ₁t) + c₂v₂e^(λ₂t)   │
  └────────────────────────────────────┘
```

---

## 2. Case 1: Distinct Real Eigenvalues

If $\lambda_1 \neq \lambda_2$ are real with eigenvectors $\mathbf{v}_1, \mathbf{v}_2$:

$$\boxed{\mathbf{x}(t) = c_1 \mathbf{v}_1 e^{\lambda_1 t} + c_2 \mathbf{v}_2 e^{\lambda_2 t}}$$

### Example 1

$$\mathbf{x}' = \begin{pmatrix} 1 & 3 \\ 5 & 3 \end{pmatrix}\mathbf{x}$$

**Characteristic equation**:

$$\det\begin{pmatrix} 1-\lambda & 3 \\ 5 & 3-\lambda \end{pmatrix} = (1-\lambda)(3-\lambda) - 15 = \lambda^2 - 4\lambda - 12 = 0$$

$$(\lambda - 6)(\lambda + 2) = 0 \implies \lambda_1 = 6, \quad \lambda_2 = -2$$

**For $\lambda_1 = 6$**: $(A - 6I)\mathbf{v} = 0$

$$\begin{pmatrix} -5 & 3 \\ 5 & -3 \end{pmatrix}\mathbf{v} = 0 \implies \mathbf{v}_1 = \begin{pmatrix} 3 \\ 5 \end{pmatrix}$$

**For $\lambda_2 = -2$**: $(A + 2I)\mathbf{v} = 0$

$$\begin{pmatrix} 3 & 3 \\ 5 & 5 \end{pmatrix}\mathbf{v} = 0 \implies \mathbf{v}_2 = \begin{pmatrix} 1 \\ -1 \end{pmatrix}$$

$$\boxed{\mathbf{x}(t) = c_1 \begin{pmatrix} 3 \\ 5 \end{pmatrix}e^{6t} + c_2 \begin{pmatrix} 1 \\ -1 \end{pmatrix}e^{-2t}}$$

---

## 3. Case 2: Complex Eigenvalues

If $\lambda = \alpha \pm i\beta$ with eigenvector $\mathbf{v} = \mathbf{a} + i\mathbf{b}$:

$$\mathbf{x}_1 = e^{\alpha t}(\mathbf{a}\cos\beta t - \mathbf{b}\sin\beta t)$$
$$\mathbf{x}_2 = e^{\alpha t}(\mathbf{a}\sin\beta t + \mathbf{b}\cos\beta t)$$

$$\boxed{\mathbf{x}(t) = c_1\mathbf{x}_1 + c_2\mathbf{x}_2}$$

### Example 2

$$\mathbf{x}' = \begin{pmatrix} 0 & 1 \\ -1 & 0 \end{pmatrix}\mathbf{x}$$

$\lambda^2 + 1 = 0 \implies \lambda = \pm i$

For $\lambda = i$:

$$\begin{pmatrix} -i & 1 \\ -1 & -i \end{pmatrix}\mathbf{v} = 0 \implies \mathbf{v} = \begin{pmatrix} 1 \\ i \end{pmatrix} = \begin{pmatrix} 1 \\ 0 \end{pmatrix} + i\begin{pmatrix} 0 \\ 1 \end{pmatrix}$$

So $\mathbf{a} = \begin{pmatrix} 1 \\ 0 \end{pmatrix}$, $\mathbf{b} = \begin{pmatrix} 0 \\ 1 \end{pmatrix}$, and $\alpha = 0$, $\beta = 1$:

$$\mathbf{x}(t) = c_1\begin{pmatrix} \cos t \\ -\sin t \end{pmatrix} + c_2\begin{pmatrix} \sin t \\ \cos t \end{pmatrix}$$

---

## 4. Case 3: Repeated Eigenvalues

If $\lambda_1 = \lambda_2 = \lambda$ (multiplicity 2):

### Subcase 3a: Two independent eigenvectors

$$\mathbf{x}(t) = c_1\mathbf{v}_1 e^{\lambda t} + c_2\mathbf{v}_2 e^{\lambda t}$$

### Subcase 3b: Only one eigenvector (deficient case)

Need a **generalized eigenvector** $\mathbf{w}$ satisfying $(A - \lambda I)\mathbf{w} = \mathbf{v}$

$$\boxed{\mathbf{x}(t) = c_1\mathbf{v}e^{\lambda t} + c_2(\mathbf{v}t + \mathbf{w})e^{\lambda t}}$$

### Example 3

$$\mathbf{x}' = \begin{pmatrix} 3 & 1 \\ -1 & 5 \end{pmatrix}\mathbf{x}$$

$\lambda^2 - 8\lambda + 16 = 0 \implies \lambda = 4$ (repeated)

For $\lambda = 4$: $(A - 4I)\mathbf{v} = 0$

$$\begin{pmatrix} -1 & 1 \\ -1 & 1 \end{pmatrix}\mathbf{v} = 0 \implies \mathbf{v} = \begin{pmatrix} 1 \\ 1 \end{pmatrix}$$

Only one eigenvector. Find generalized eigenvector $(A - 4I)\mathbf{w} = \mathbf{v}$:

$$\begin{pmatrix} -1 & 1 \\ -1 & 1 \end{pmatrix}\mathbf{w} = \begin{pmatrix} 1 \\ 1 \end{pmatrix} \implies -w_1 + w_2 = 1 \implies \mathbf{w} = \begin{pmatrix} 0 \\ 1 \end{pmatrix}$$

$$\boxed{\mathbf{x}(t) = c_1\begin{pmatrix} 1 \\ 1 \end{pmatrix}e^{4t} + c_2\left[\begin{pmatrix} 1 \\ 1 \end{pmatrix}t + \begin{pmatrix} 0 \\ 1 \end{pmatrix}\right]e^{4t}}$$

---

## 5. Non-Homogeneous Systems

For $\mathbf{x}' = A\mathbf{x} + \mathbf{g}(t)$:

### Method: Variation of Parameters

$$\mathbf{x}_p = \Phi(t)\int \Phi^{-1}(t)\,\mathbf{g}(t)\,dt$$

where $\Phi(t)$ is the **fundamental matrix** (columns are the independent homogeneous solutions).

### Method: Undetermined Coefficients

Works when $\mathbf{g}(t)$ has polynomial, exponential, or sinusoidal entries.

---

## 6. Matrix Exponential

The solution of $\mathbf{x}' = A\mathbf{x}$, $\mathbf{x}(0) = \mathbf{x}_0$ is:

$$\mathbf{x}(t) = e^{At}\mathbf{x}_0$$

where the **matrix exponential** is:

$$e^{At} = I + At + \frac{(At)^2}{2!} + \frac{(At)^3}{3!} + \cdots$$

For diagonalizable $A = PDP^{-1}$:

$$e^{At} = Pe^{Dt}P^{-1} = P\begin{pmatrix} e^{\lambda_1 t} & 0 \\ 0 & e^{\lambda_2 t} \end{pmatrix}P^{-1}$$

---

## 7. Summary of Cases

```
  EIGENVALUE DECISION TREE

  ┌────────────────────────┐
  │ Find eigenvalues of A  │
  └───────────┬────────────┘
              ▼
     ┌────────┴────────┐
     ▼                  ▼
  Distinct?          Repeated?
     │                  │
  ┌──┴──┐           ┌──┴──┐
  ▼      ▼           ▼     ▼
 Real  Complex    2 ind.  1 ind.
  │      │       eigvec  eigvec
  │      │         │       │
  ▼      ▼         ▼       ▼
 v·eλt  Euler   v₁eλt   v·eλt +
         form.  v₂eλt   (vt+w)eλt
```

---

## Summary Table

| Case | Eigenvalues | Solution Form |
|------|------------|---------------|
| Distinct real | $\lambda_1 \neq \lambda_2$, both real | $c_1\mathbf{v}_1 e^{\lambda_1 t} + c_2\mathbf{v}_2 e^{\lambda_2 t}$ |
| Complex | $\alpha \pm i\beta$ | $e^{\alpha t}(c_1\cos\beta t + c_2\sin\beta t)$ terms |
| Repeated, 2 eigenvectors | $\lambda = \lambda$ | $c_1\mathbf{v}_1 e^{\lambda t} + c_2\mathbf{v}_2 e^{\lambda t}$ |
| Repeated, 1 eigenvector | $\lambda = \lambda$ | $c_1\mathbf{v}e^{\lambda t} + c_2(\mathbf{v}t + \mathbf{w})e^{\lambda t}$ |
| Non-homogeneous | Any | $\mathbf{x}_h + \mathbf{x}_p$ |
| Matrix exponential | $A = PDP^{-1}$ | $e^{At} = Pe^{Dt}P^{-1}$ |

---

## Quick Revision Questions

1. Find the general solution of $\mathbf{x}' = \begin{pmatrix} 2 & 1 \\ 0 & 3 \end{pmatrix}\mathbf{x}$.

2. Solve $\mathbf{x}' = \begin{pmatrix} 0 & -2 \\ 2 & 0 \end{pmatrix}\mathbf{x}$ and describe the trajectory shapes.

3. For $A = \begin{pmatrix} 5 & -4 \\ 1 & 1 \end{pmatrix}$, find the eigenvalues and determine which case applies. Solve the system.

4. Explain why complex eigenvalues always appear in conjugate pairs for real matrices.

5. Compute $e^{At}$ for $A = \begin{pmatrix} 0 & 1 \\ 0 & 0 \end{pmatrix}$.

6. Solve $\mathbf{x}' = \begin{pmatrix} 1 & 1 \\ 4 & 1 \end{pmatrix}\mathbf{x}$, $\mathbf{x}(0) = \begin{pmatrix} 1 \\ 6 \end{pmatrix}$.

---

[← Previous: Introduction & First-Order Systems](01-introduction-and-first-order-systems.md) | [Back to README](../README.md) | [Next: Phase Plane Analysis →](03-phase-plane.md)
