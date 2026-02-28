# 5.4 Applications of Diagonalization — Matrix Powers

[← Previous: Diagonalization Procedure](03-diagonalization-procedure.md) · [Back to README](../README.md) · [Next: Non-Diagonalizable Matrices →](05-non-diagonalizable-matrices.md)

---

## Chapter Overview

The payoff of diagonalisation: computing matrix powers $A^k$, solving recurrence relations (Fibonacci!), and solving systems of differential equations — all become trivial with $A = PDP^{-1}$.

---

## 1. Matrix Powers via Diagonalisation

If $A = PDP^{-1}$, then:

$$A^k = PD^kP^{-1}$$

where $D^k = \text{diag}(\lambda_1^k, \lambda_2^k, \dots, \lambda_n^k)$.

**Why it works:**

$$A^2 = (PDP^{-1})(PDP^{-1}) = PD(P^{-1}P)DP^{-1} = PD^2P^{-1}$$

By induction: $A^k = PD^kP^{-1}$.

```
  Computing A¹⁰⁰:
  
  Direct:  A × A × A × ··· × A    (100 multiplications!)
  
  Via diag:  P · diag(λ₁¹⁰⁰, λ₂¹⁰⁰, ...) · P⁻¹   (just raise scalars!)
```

---

## 2. Worked Example: Computing $A^k$

$$A = \begin{pmatrix} 4 & 1 \\ 2 & 3 \end{pmatrix}$$

From earlier: $\lambda_1 = 5, \lambda_2 = 2$

$$P = \begin{pmatrix} 1 & 1 \\ 1 & -2 \end{pmatrix}, \quad P^{-1} = \frac{1}{-3}\begin{pmatrix} -2 & -1 \\ -1 & 1 \end{pmatrix} = \begin{pmatrix} 2/3 & 1/3 \\ 1/3 & -1/3 \end{pmatrix}$$

$$A^k = P \begin{pmatrix} 5^k & 0 \\ 0 & 2^k \end{pmatrix} P^{-1} = \frac{1}{3}\begin{pmatrix} 2 \cdot 5^k + 2^k & 5^k - 2^k \\ 2 \cdot 5^k - 2 \cdot 2^k & 5^k + 2 \cdot 2^k \end{pmatrix}$$

**Check** ($k = 1$): $\frac{1}{3}\begin{pmatrix} 12 & 3 \\ 6 & 9 \end{pmatrix} = \begin{pmatrix} 4 & 1 \\ 2 & 3 \end{pmatrix}$ ✓

---

## 3. The Fibonacci Sequence

The Fibonacci sequence: $F_0 = 0, F_1 = 1, F_{n+1} = F_n + F_{n-1}$.

Write as a matrix recurrence:

$$\begin{pmatrix} F_{n+1} \\ F_n \end{pmatrix} = \begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix} \begin{pmatrix} F_n \\ F_{n-1} \end{pmatrix}$$

So $\begin{pmatrix} F_{n+1} \\ F_n \end{pmatrix} = A^n \begin{pmatrix} 1 \\ 0 \end{pmatrix}$ where $A = \begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix}$

**Diagonalise $A$:**

$p(\lambda) = \lambda^2 - \lambda - 1 = 0 \implies \lambda = \frac{1 \pm \sqrt{5}}{2}$

$\phi = \frac{1 + \sqrt{5}}{2} \approx 1.618$ (golden ratio), $\psi = \frac{1 - \sqrt{5}}{2} \approx -0.618$

**Result (Binet's formula):**

$$\boxed{F_n = \frac{\phi^n - \psi^n}{\sqrt{5}} = \frac{1}{\sqrt{5}}\left[\left(\frac{1+\sqrt{5}}{2}\right)^n - \left(\frac{1-\sqrt{5}}{2}\right)^n\right]}$$

Since $|\psi| < 1$, for large $n$: $F_n \approx \frac{\phi^n}{\sqrt{5}}$.

---

## 4. Systems of Linear Differential Equations

Consider $\mathbf{x}'(t) = A\mathbf{x}(t)$ where $A$ is diagonalisable.

Let $\mathbf{x} = P\mathbf{y}$. Then:

$$P\mathbf{y}' = AP\mathbf{y} \implies \mathbf{y}' = P^{-1}AP\mathbf{y} = D\mathbf{y}$$

The decoupled system: $y_i' = \lambda_i y_i \implies y_i(t) = c_i e^{\lambda_i t}$

**Solution:** $\mathbf{x}(t) = P\mathbf{y}(t) = c_1 e^{\lambda_1 t}\mathbf{v}_1 + \dots + c_n e^{\lambda_n t}\mathbf{v}_n$

```
  Coupled system        Decoupled system
  x' = Ax               y' = Dy
  
  ┌──┐   ┌──┐ ┌──┐      ┌──┐ ┌────┐ ┌──┐
  │x₁│   │  │ │x₁│      │y₁│ │λ₁  │ │y₁│
  │  │ = │A │ │  │  →→→  │  │=│  λ₂│ │  │
  │x₂│   │  │ │x₂│      │y₂│ │    │ │y₂│
  └──┘   └──┘ └──┘      └──┘ └────┘ └──┘
  
  (coupled)              (independent!)
  
  y₁' = λ₁y₁ → y₁ = c₁eᵏ¹ᵗ
  y₂' = λ₂y₂ → y₂ = c₂eᵏ²ᵗ
```

### Worked Example

$$\mathbf{x}' = \begin{pmatrix} -3 & 1 \\ 2 & -2 \end{pmatrix}\mathbf{x}$$

$p(\lambda) = \lambda^2 + 5\lambda + 4 = (\lambda + 1)(\lambda + 4)$

$\lambda_1 = -1$: $\mathbf{v}_1 = (1, 2)^T$ $\qquad$ $\lambda_2 = -4$: $\mathbf{v}_2 = (1, -1)^T$

**Solution:**

$$\mathbf{x}(t) = c_1 e^{-t}\begin{pmatrix} 1 \\ 2 \end{pmatrix} + c_2 e^{-4t}\begin{pmatrix} 1 \\ -1 \end{pmatrix}$$

Both eigenvalues negative → solution decays to $\mathbf{0}$ (stable equilibrium).

---

## 5. Matrix Exponential

> **Definition.** $e^A = I + A + \frac{A^2}{2!} + \frac{A^3}{3!} + \dots$

If $A = PDP^{-1}$:

$$e^A = Pe^DP^{-1} = P \,\text{diag}(e^{\lambda_1}, e^{\lambda_2}, \dots, e^{\lambda_n})\, P^{-1}$$

This solves $\mathbf{x}' = A\mathbf{x}$ with $\mathbf{x}(0) = \mathbf{x}_0$:

$$\mathbf{x}(t) = e^{At}\mathbf{x}_0$$

---

## 6. Long-Term Behaviour from Eigenvalues

| Eigenvalue condition | Behaviour of $A^k$ as $k \to \infty$ |
|---|---|
| All $|\lambda_i| < 1$ | $A^k \to 0$ (stable) |
| All $|\lambda_i| \leq 1$, some = 1 | $A^k$ bounded |
| Some $|\lambda_i| > 1$ | $A^k$ diverges (unstable) |
| All $\lambda_i > 0$ | Powers stay positive |

| Eigenvalue condition | Behaviour of $e^{At}$ as $t \to \infty$ |
|---|---|
| All $\text{Re}(\lambda_i) < 0$ | $e^{At} \to 0$ (asymptotically stable) |
| Some $\text{Re}(\lambda_i) > 0$ | Solution grows (unstable) |

---

## Summary Table

| Application | Formula |
|-------------|---------|
| Matrix powers | $A^k = PD^kP^{-1}$ |
| Recurrence relations | $\mathbf{x}_{n+1} = A\mathbf{x}_n \implies \mathbf{x}_n = A^n\mathbf{x}_0$ |
| Fibonacci | $F_n = \frac{\phi^n - \psi^n}{\sqrt{5}}$ |
| ODEs ($\mathbf{x}' = A\mathbf{x}$) | $\mathbf{x}(t) = \sum c_i e^{\lambda_i t}\mathbf{v}_i$ |
| Matrix exponential | $e^{A} = Pe^{D}P^{-1}$ |
| Stability (discrete) | $A^k \to 0$ iff all $|\lambda_i| < 1$ |
| Stability (continuous) | $e^{At} \to 0$ iff all $\text{Re}(\lambda_i) < 0$ |

---

## Quick Revision Questions

1. If $A$ has eigenvalues $2$ and $-1$, does $A^k$ converge as $k \to \infty$?

2. Derive Binet's formula for Fibonacci numbers using matrix diagonalisation.

3. Solve $\mathbf{x}' = \begin{pmatrix} 0 & 1 \\ -2 & 3 \end{pmatrix}\mathbf{x}$ using diagonalisation.

4. If $A = PDP^{-1}$, show that $e^A = Pe^DP^{-1}$.

5. Compute $A^{50}$ for $A = \begin{pmatrix} 1 & 1 \\ 0 & 2 \end{pmatrix}$.

---

[← Previous: Diagonalization Procedure](03-diagonalization-procedure.md) · [Back to README](../README.md) · [Next: Non-Diagonalizable Matrices →](05-non-diagonalizable-matrices.md)
