# 📐 Numerical Methods — Comprehensive Study Notes

> **A complete theory-driven guide to computational mathematics, covering error analysis, root-finding, interpolation, differentiation, integration, ODEs, PDEs, and more.**

---

## Course Overview

Numerical Methods is the study of algorithms that use numerical approximation for the problems of mathematical analysis. Unlike symbolic methods that seek exact closed-form solutions, numerical methods produce approximate solutions within controlled error bounds. These techniques are indispensable in engineering, physics, finance, and computer science — wherever analytical solutions are impractical or impossible.

### What You'll Learn

| Area | Key Skills |
|------|-----------|
| **Error Analysis** | Quantify, propagate, and control computational errors |
| **Root Finding** | Solve nonlinear equations iteratively |
| **Linear Systems** | Solve large systems using direct and iterative methods |
| **Interpolation** | Construct approximating polynomials from data |
| **Differentiation** | Approximate derivatives from discrete data |
| **Integration** | Compute definite integrals numerically |
| **ODEs & PDEs** | Solve differential equations step-by-step |
| **Eigenvalues** | Find dominant eigenvalues iteratively |
| **Curve Fitting** | Fit models to experimental data |

### Prerequisites

- Calculus (differentiation, integration, Taylor series)
- Linear algebra (matrices, determinants, systems of equations)
- Basic programming concepts (helpful but not mandatory)

---

## 📚 Complete Table of Contents

### Unit 1: Errors and Approximations
| # | Topic | Link |
|---|-------|------|
| 1.1 | Types of Errors (Absolute, Relative, Percentage) | [01-types-of-errors.md](01-Errors-and-Approximations/01-types-of-errors.md) |
| 1.2 | Round-off and Truncation Errors | [02-roundoff-and-truncation-errors.md](01-Errors-and-Approximations/02-roundoff-and-truncation-errors.md) |
| 1.3 | Significant Figures | [03-significant-figures.md](01-Errors-and-Approximations/03-significant-figures.md) |
| 1.4 | Error Propagation | [04-error-propagation.md](01-Errors-and-Approximations/04-error-propagation.md) |
| 1.5 | Stability of Algorithms | [05-stability-of-algorithms.md](01-Errors-and-Approximations/05-stability-of-algorithms.md) |

### Unit 2: Solution of Equations
| # | Topic | Link |
|---|-------|------|
| 2.1 | Bisection Method | [01-bisection-method.md](02-Solution-of-Equations/01-bisection-method.md) |
| 2.2 | Regula Falsi (False Position) | [02-regula-falsi.md](02-Solution-of-Equations/02-regula-falsi.md) |
| 2.3 | Newton-Raphson Method | [03-newton-raphson.md](02-Solution-of-Equations/03-newton-raphson.md) |
| 2.4 | Secant Method | [04-secant-method.md](02-Solution-of-Equations/04-secant-method.md) |
| 2.5 | Fixed-Point Iteration | [05-fixed-point-iteration.md](02-Solution-of-Equations/05-fixed-point-iteration.md) |
| 2.6 | Convergence Analysis | [06-convergence-analysis.md](02-Solution-of-Equations/06-convergence-analysis.md) |

### Unit 3: Solution of System of Equations
| # | Topic | Link |
|---|-------|------|
| 3.1 | Gaussian Elimination | [01-gaussian-elimination.md](03-System-of-Equations/01-gaussian-elimination.md) |
| 3.2 | Gauss-Jordan Method | [02-gauss-jordan.md](03-System-of-Equations/02-gauss-jordan.md) |
| 3.3 | LU Decomposition | [03-lu-decomposition.md](03-System-of-Equations/03-lu-decomposition.md) |
| 3.4 | Jacobi Method | [04-jacobi-method.md](03-System-of-Equations/04-jacobi-method.md) |
| 3.5 | Gauss-Seidel Method | [05-gauss-seidel.md](03-System-of-Equations/05-gauss-seidel.md) |
| 3.6 | Relaxation Methods | [06-relaxation-methods.md](03-System-of-Equations/06-relaxation-methods.md) |

### Unit 4: Interpolation
| # | Topic | Link |
|---|-------|------|
| 4.1 | Finite Differences | [01-finite-differences.md](04-Interpolation/01-finite-differences.md) |
| 4.2 | Newton's Forward Difference | [02-newton-forward.md](04-Interpolation/02-newton-forward.md) |
| 4.3 | Newton's Backward Difference | [03-newton-backward.md](04-Interpolation/03-newton-backward.md) |
| 4.4 | Lagrange Interpolation | [04-lagrange-interpolation.md](04-Interpolation/04-lagrange-interpolation.md) |
| 4.5 | Newton's Divided Difference | [05-newton-divided-difference.md](04-Interpolation/05-newton-divided-difference.md) |
| 4.6 | Inverse Interpolation | [06-inverse-interpolation.md](04-Interpolation/06-inverse-interpolation.md) |
| 4.7 | Spline Interpolation | [07-spline-interpolation.md](04-Interpolation/07-spline-interpolation.md) |

### Unit 5: Numerical Differentiation
| # | Topic | Link |
|---|-------|------|
| 5.1 | Forward Difference Formula | [01-forward-difference.md](05-Numerical-Differentiation/01-forward-difference.md) |
| 5.2 | Backward Difference Formula | [02-backward-difference.md](05-Numerical-Differentiation/02-backward-difference.md) |
| 5.3 | Central Difference Formula | [03-central-difference.md](05-Numerical-Differentiation/03-central-difference.md) |
| 5.4 | Higher Order Derivatives | [04-higher-order-derivatives.md](05-Numerical-Differentiation/04-higher-order-derivatives.md) |
| 5.5 | Error Estimation | [05-error-estimation.md](05-Numerical-Differentiation/05-error-estimation.md) |

### Unit 6: Numerical Integration
| # | Topic | Link |
|---|-------|------|
| 6.1 | Trapezoidal Rule | [01-trapezoidal-rule.md](06-Numerical-Integration/01-trapezoidal-rule.md) |
| 6.2 | Simpson's 1/3 Rule | [02-simpsons-one-third.md](06-Numerical-Integration/02-simpsons-one-third.md) |
| 6.3 | Simpson's 3/8 Rule | [03-simpsons-three-eighth.md](06-Numerical-Integration/03-simpsons-three-eighth.md) |
| 6.4 | Weddle's Rule | [04-weddles-rule.md](06-Numerical-Integration/04-weddles-rule.md) |
| 6.5 | Gaussian Quadrature | [05-gaussian-quadrature.md](06-Numerical-Integration/05-gaussian-quadrature.md) |
| 6.6 | Romberg Integration | [06-romberg-integration.md](06-Numerical-Integration/06-romberg-integration.md) |
| 6.7 | Error Analysis | [07-error-analysis.md](06-Numerical-Integration/07-error-analysis.md) |

### Unit 7: Ordinary Differential Equations
| # | Topic | Link |
|---|-------|------|
| 7.1 | Euler's Method | [01-eulers-method.md](07-Ordinary-Differential-Equations/01-eulers-method.md) |
| 7.2 | Modified Euler's Method | [02-modified-eulers.md](07-Ordinary-Differential-Equations/02-modified-eulers.md) |
| 7.3 | Runge-Kutta Methods (RK2, RK4) | [03-runge-kutta.md](07-Ordinary-Differential-Equations/03-runge-kutta.md) |
| 7.4 | Predictor-Corrector Methods | [04-predictor-corrector.md](07-Ordinary-Differential-Equations/04-predictor-corrector.md) |
| 7.5 | Milne's Method | [05-milnes-method.md](07-Ordinary-Differential-Equations/05-milnes-method.md) |
| 7.6 | Adams-Bashforth-Moulton Method | [06-adams-bashforth-moulton.md](07-Ordinary-Differential-Equations/06-adams-bashforth-moulton.md) |

### Unit 8: Eigenvalue Problems
| # | Topic | Link |
|---|-------|------|
| 8.1 | Power Method | [01-power-method.md](08-Eigenvalue-Problems/01-power-method.md) |
| 8.2 | Inverse Power Method | [02-inverse-power-method.md](08-Eigenvalue-Problems/02-inverse-power-method.md) |
| 8.3 | QR Algorithm (Introduction) | [03-qr-algorithm.md](08-Eigenvalue-Problems/03-qr-algorithm.md) |

### Unit 9: Curve Fitting
| # | Topic | Link |
|---|-------|------|
| 9.1 | Least Squares Method | [01-least-squares.md](09-Curve-Fitting/01-least-squares.md) |
| 9.2 | Linear Regression | [02-linear-regression.md](09-Curve-Fitting/02-linear-regression.md) |
| 9.3 | Polynomial Fitting | [03-polynomial-fitting.md](09-Curve-Fitting/03-polynomial-fitting.md) |
| 9.4 | Exponential Fitting | [04-exponential-fitting.md](09-Curve-Fitting/04-exponential-fitting.md) |

### Unit 10: Partial Differential Equations
| # | Topic | Link |
|---|-------|------|
| 10.1 | Classification of PDEs | [01-classification-of-pdes.md](10-Partial-Differential-Equations/01-classification-of-pdes.md) |
| 10.2 | Finite Difference Methods | [02-finite-difference-methods.md](10-Partial-Differential-Equations/02-finite-difference-methods.md) |
| 10.3 | Laplace Equation | [03-laplace-equation.md](10-Partial-Differential-Equations/03-laplace-equation.md) |
| 10.4 | Heat Equation | [04-heat-equation.md](10-Partial-Differential-Equations/04-heat-equation.md) |
| 10.5 | Wave Equation | [05-wave-equation.md](10-Partial-Differential-Equations/05-wave-equation.md) |

---

## How to Use These Notes

1. **Sequential Study**: Follow units in order — each builds on the previous.
2. **Quick Revision**: Jump to summary tables at the end of each chapter.
3. **Practice**: Work through solved examples before attempting revision questions.
4. **Navigation**: Use Previous/Next links at the bottom of each chapter file.

---

## Quick Reference: Key Formulas

| Method | Formula | Order |
|--------|---------|-------|
| Bisection | $c = \frac{a+b}{2}$ | $O(\log_2(\frac{b-a}{\epsilon}))$ |
| Newton-Raphson | $x_{n+1} = x_n - \frac{f(x_n)}{f'(x_n)}$ | Quadratic |
| Trapezoidal Rule | $\int_a^b f(x)dx \approx \frac{h}{2}[f(a) + f(b)]$ | $O(h^2)$ |
| Simpson's 1/3 | $\int_a^b f(x)dx \approx \frac{h}{3}[f_0 + 4f_1 + f_2]$ | $O(h^4)$ |
| Euler's Method | $y_{n+1} = y_n + hf(x_n, y_n)$ | $O(h)$ |
| RK4 | $y_{n+1} = y_n + \frac{h}{6}(k_1+2k_2+2k_3+k_4)$ | $O(h^4)$ |

---

*Last Updated: February 2026*
