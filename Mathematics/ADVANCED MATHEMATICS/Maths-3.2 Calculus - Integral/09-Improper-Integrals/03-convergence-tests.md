# Chapter 3: Convergence Tests for Improper Integrals

[← Previous: Type II — Discontinuous Integrands](02-type-II-discontinuous-integrand.md) | [Back to README](../README.md) | [Next: Comparison Tests →](04-comparison-tests.md)

---

## 3.1 Overview

When we can't evaluate an improper integral directly, we need tools to determine whether it converges or diverges. This chapter covers the main convergence tests.

---

## 3.2 Direct Comparison Test

> **Theorem:** Suppose 0 ≤ f(x) ≤ g(x) for all x ≥ a.
>
> 1. If ∫ₐ^∞ g(x) dx **converges**, then ∫ₐ^∞ f(x) dx **converges**.
> 2. If ∫ₐ^∞ f(x) dx **diverges**, then ∫ₐ^∞ g(x) dx **diverges**.

```
  "Smaller than convergent → convergent"
  "Bigger than divergent → divergent"
  
  g(x) ─╲                
         ╲    ← g converges
  f(x) ──╲───────
           ╲──── ← f ≤ g, so f also converges
  ──────────────── x
```

### How to Use

1. Compare f(x) to a known function g(x)
2. Common comparisons: 1/xᵖ, e⁻ˣ, 1/(x² + 1)
3. Ensure the inequality holds for all x in the region

---

## 3.3 Limit Comparison Test

> **Theorem:** If f(x) > 0 and g(x) > 0 for x ≥ a, and:
>
> $$L = \lim_{x \to \infty} \frac{f(x)}{g(x)}$$
>
> - If 0 < L < ∞: f and g both converge or both diverge.
> - If L = 0 and ∫g converges: ∫f converges.
> - If L = ∞ and ∫g diverges: ∫f diverges.

This is often easier than direct comparison because you don't need a precise inequality — just asymptotic behavior.

### Example

$$\int_1^{\infty} \frac{x+3}{x^3-1}\, dx$$

Compare with 1/x²: $\lim_{x\to\infty} \frac{(x+3)/(x^3-1)}{1/x^2} = \lim_{x\to\infty} \frac{x^3+3x^2}{x^3-1} = 1$

Since 0 < 1 < ∞ and ∫ 1/x² converges, the original integral **converges**. ✓

---

## 3.4 Absolute Convergence

> **Definition:** ∫ₐ^∞ f(x) dx converges **absolutely** if ∫ₐ^∞ |f(x)| dx converges.
>
> **Theorem:** Absolute convergence implies convergence.

$$\int_a^{\infty} |f(x)|\, dx \text{ converges} \implies \int_a^{\infty} f(x)\, dx \text{ converges}$$

An integral that converges but not absolutely converges **conditionally**.

### Example: Conditional Convergence

$$\int_1^{\infty} \frac{\sin x}{x}\, dx \quad \text{converges (conditionally)}$$

$$\int_1^{\infty} \frac{|\sin x|}{x}\, dx \quad \text{diverges}$$

---

## 3.5 Dirichlet's Test

> **Theorem:** ∫ₐ^∞ f(x)g(x) dx converges if:
> 1. F(t) = ∫ₐᵗ f(x) dx is bounded for all t
> 2. g(x) is monotonically decreasing to 0

### Application

∫₁^∞ (sin x)/x dx: Let f(x) = sin x (bounded integral), g(x) = 1/x (decreasing to 0). ✓

---

## 3.6 Quick Reference — Which Test to Use

```
  CONVERGENCE TEST SELECTION
  ══════════════════════════
  
  Can you evaluate the integral directly?
  ├── YES → Do it! No test needed.
  └── NO ↓
  
  Is the integrand always positive (or always negative)?
  ├── YES → Comparison or Limit Comparison test
  │         Compare with 1/xᵖ or e⁻ᵃˣ  
  └── NO → Integrand changes sign
           ├── Check absolute convergence (∫|f|)
           │   └── If |f| converges → f converges absolutely
           └── If |f| diverges → try Dirichlet's test
               or Abel's test for conditional convergence
```

---

## 3.7 Standard Comparison Functions

| Test Function | Behavior | Use When |
|---|---|---|
| 1/xᵖ (p > 1) | Converges at ∞ | Rational functions |
| 1/xᵖ (p < 1) | Converges at 0 | Singularity at origin |
| e⁻ᵃˣ (a > 0) | Converges at ∞ | Exponential decay |
| 1/(x ln x)^p | Converges at ∞ if p > 1 | Logarithmic factors |
| 1/x | DIVERGES at ∞ | Lower bound for divergence |

---

## 3.8 Worked Examples

### Example 1: Direct Comparison

$$\int_1^{\infty} \frac{1}{x^2 + x}\, dx$$

Since x² + x > x² for x > 0: $\frac{1}{x^2+x} < \frac{1}{x^2}$

∫₁^∞ 1/x² converges → ∫₁^∞ 1/(x²+x) **converges** by direct comparison.

---

### Example 2: Limit Comparison

$$\int_2^{\infty} \frac{\sqrt{x}}{x^2 - 1}\, dx$$

For large x: $\frac{\sqrt{x}}{x^2 - 1} \approx \frac{x^{1/2}}{x^2} = \frac{1}{x^{3/2}}$

$$\lim_{x\to\infty}\frac{\sqrt{x}/(x^2-1)}{1/x^{3/2}} = \lim_{x\to\infty}\frac{x^2}{x^2-1} = 1$$

Since ∫ 1/x^{3/2} converges (p = 3/2 > 1), the integral **converges**.

---

### Example 3: Using Absolute Convergence

$$\int_1^{\infty} \frac{\cos x}{x^2}\, dx$$

$$\left|\frac{\cos x}{x^2}\right| \leq \frac{1}{x^2}$$

∫₁^∞ 1/x² converges → ∫ |cos x/x²| converges → ∫ cos x/x² **converges absolutely**.

---

## Summary Table

| Test | Condition | Conclusion |
|---|---|---|
| Direct Comparison | 0 ≤ f ≤ g, ∫g conv | ∫f converges |
| Direct Comparison | f ≥ g ≥ 0, ∫g div | ∫f diverges |
| Limit Comparison | lim f/g = L (0 < L < ∞) | Same behavior as ∫g |
| Absolute Convergence | ∫\|f\| converges | ∫f converges |
| Dirichlet's Test | ∫f bounded, g↓0 | ∫fg converges |
| p-test at ∞ | ∫₁^∞ 1/xᵖ | p>1: conv, p≤1: div |
| p-test at 0 | ∫₀¹ 1/xᵖ | p<1: conv, p≥1: div |

---

## Quick Revision Questions

1. Use the comparison test to determine if ∫₁^∞ 1/(x³+1) dx converges.

2. Use the limit comparison test to determine if ∫₁^∞ (3x+2)/(x⁴+x²+1) dx converges.

3. Does ∫₁^∞ sin²(x)/x² dx converge? Justify with a specific test.

4. Explain the difference between absolute and conditional convergence with an example.

5. Determine convergence of ∫₀¹ 1/√(x − x²) dx. (Hint: check behavior at both endpoints.)

6. Use Dirichlet's test to show that ∫₁^∞ sin(x)/x dx converges.

---

[← Previous: Type II — Discontinuous Integrands](02-type-II-discontinuous-integrand.md) | [Back to README](../README.md) | [Next: Comparison Tests →](04-comparison-tests.md)
