# Chapter 3: Partial Fractions

[← Previous: Integration by Parts](02-integration-by-parts.md) | [Back to README](../README.md) | [Next: Trigonometric Substitutions →](04-trigonometric-substitutions.md)

---

## 3.1 Overview

**Partial fraction decomposition** is a technique for integrating **rational functions** — ratios of polynomials P(x)/Q(x). The idea is to decompose a complex fraction into a sum of simpler fractions that can be integrated individually using standard formulas.

---

## 3.2 When to Use Partial Fractions

```
  Given: ∫ P(x)/Q(x) dx    where P, Q are polynomials
     │
     ├── Is degree(P) ≥ degree(Q)?
     │       │
     │      Yes → Perform polynomial long division first
     │       │     to get: Quotient + Remainder/Q(x)
     │       │     then decompose Remainder/Q(x)
     │       │
     │      No → Proceed directly to decomposition
     │
     ▼
  Factor Q(x) completely
     │
     ▼
  Write partial fraction form based on factor types
     │
     ▼
  Solve for unknown constants
     │
     ▼
  Integrate each simple fraction
```

---

## 3.3 Prerequisites: Polynomial Long Division

If deg(P) ≥ deg(Q), divide first.

**Example:** ∫ (x³ + 2x) / (x² + 1) dx

```
  Long division:
  
          x
  x²+1 ) x³ + 0·x² + 2x
          x³ + x
          ─────────
               x
  
  Result: x³ + 2x = (x² + 1)(x) + x
```

$$\frac{x^3 + 2x}{x^2 + 1} = x + \frac{x}{x^2 + 1}$$

$$\int \left(x + \frac{x}{x^2+1}\right) dx = \frac{x^2}{2} + \frac{1}{2}\ln(x^2+1) + C$$

---

## 3.4 Types of Partial Fraction Decomposition

### Case 1: Distinct Linear Factors

If Q(x) = (x − a₁)(x − a₂)···(x − aₙ), then:

$$\frac{P(x)}{Q(x)} = \frac{A_1}{x - a_1} + \frac{A_2}{x - a_2} + \cdots + \frac{A_n}{x - a_n}$$

**Example:**

$$\frac{3x + 5}{(x+1)(x+2)} = \frac{A}{x+1} + \frac{B}{x+2}$$

Multiply both sides by (x+1)(x+2):

$$3x + 5 = A(x+2) + B(x+1)$$

**Method 1 — Substitution of Roots:**
- x = −1: 3(−1) + 5 = A(1) + 0 → A = 2
- x = −2: 3(−2) + 5 = 0 + B(−1) → B = 1

$$\int \frac{3x+5}{(x+1)(x+2)}\, dx = \int \frac{2}{x+1}\, dx + \int \frac{1}{x+2}\, dx = 2\ln|x+1| + \ln|x+2| + C$$

---

### Case 2: Repeated Linear Factors

If Q(x) contains (x − a)ⁿ, use:

$$\frac{A_1}{x-a} + \frac{A_2}{(x-a)^2} + \cdots + \frac{A_n}{(x-a)^n}$$

**Example:**

$$\frac{2x + 3}{(x-1)^2} = \frac{A}{x-1} + \frac{B}{(x-1)^2}$$

Multiply by (x−1)²:

$$2x + 3 = A(x-1) + B$$

- x = 1: 5 = B → B = 5
- Compare x-coefficients: 2 = A → A = 2

$$\int \frac{2x+3}{(x-1)^2}\, dx = 2\ln|x-1| - \frac{5}{x-1} + C$$

---

### Case 3: Irreducible Quadratic Factors

If Q(x) contains (ax² + bx + c) with b² − 4ac < 0:

$$\frac{Ax + B}{ax^2 + bx + c}$$

**Example:**

$$\frac{x + 3}{x^2 + 4}  = \frac{Ax + B}{x^2 + 4}$$

Here A = 1, B = 3 (already in partial fraction form):

$$\int \frac{x + 3}{x^2 + 4}\, dx = \int \frac{x}{x^2+4}\, dx + 3\int \frac{dx}{x^2+4}$$

$$= \frac{1}{2}\ln(x^2+4) + \frac{3}{2}\tan^{-1}\left(\frac{x}{2}\right) + C$$

---

### Case 4: Repeated Irreducible Quadratic Factors

If Q(x) contains (ax² + bx + c)ⁿ:

$$\frac{A_1x + B_1}{ax^2+bx+c} + \frac{A_2x + B_2}{(ax^2+bx+c)^2} + \cdots$$

---

## 3.5 Master Decomposition Table

| Factor in Q(x) | Partial Fraction Form |
|---|---|
| (x − a) | A/(x − a) |
| (x − a)² | A/(x−a) + B/(x−a)² |
| (x − a)³ | A/(x−a) + B/(x−a)² + C/(x−a)³ |
| (x² + bx + c), irred. | (Ax + B)/(x² + bx + c) |
| (x² + bx + c)², irred. | (Ax+B)/(x²+bx+c) + (Cx+D)/(x²+bx+c)² |

---

## 3.6 Methods for Finding Constants

### Method 1: Substitution of Roots (Cover-up Method)

Substitute the roots of the denominator to find constants directly.

### Method 2: Equating Coefficients

Expand the right side and match coefficients of each power of x.

### Method 3: Strategic x-Values

Choose convenient values of x (not necessarily roots) to generate equations.

### Comparison of Methods

```
  Substitution          Equating Coefficients     Strategic Values
  ─────────────         ─────────────────────     ─────────────────
  ✓ Fast for            ✓ Works for all cases     ✓ Flexible
    linear factors      ✓ Systematic              ✓ Combine with
  ✗ Doesn't work        ✗ Can be tedious            above methods
    for quadratic         for many terms
    factors directly
```

---

## 3.7 Complete Worked Example

$$\int \frac{5x^2 + 3x + 1}{(x+1)(x^2+1)}\, dx$$

**Step 1:** Set up decomposition:

$$\frac{5x^2 + 3x + 1}{(x+1)(x^2+1)} = \frac{A}{x+1} + \frac{Bx + C}{x^2+1}$$

**Step 2:** Multiply by (x+1)(x²+1):

$$5x^2 + 3x + 1 = A(x^2 + 1) + (Bx + C)(x + 1)$$

**Step 3:** Find constants:

- x = −1: 5(1) + 3(−1) + 1 = A(2) + 0 → 3 = 2A → A = 3/2

Expand right side:

$$A(x^2+1) + Bx^2 + Bx + Cx + C = (A+B)x^2 + (B+C)x + (A+C)$$

Equate coefficients:
- x²: A + B = 5 → B = 5 − 3/2 = 7/2
- x¹: B + C = 3 → C = 3 − 7/2 = −1/2
- x⁰: A + C = 1 → 3/2 − 1/2 = 1 ✓

**Step 4:** Integrate:

$$\int \frac{3/2}{x+1}\, dx + \int \frac{(7/2)x - 1/2}{x^2+1}\, dx$$

$$= \frac{3}{2}\ln|x+1| + \frac{7}{4}\ln(x^2+1) - \frac{1}{2}\tan^{-1}x + C$$

---

## 3.8 Integration Results for Partial Fractions

| Fraction Type | Integral |
|---|---|
| A/(x − a) | A ln\|x − a\| + C |
| A/(x − a)ⁿ, n > 1 | A(x−a)⁻ⁿ⁺¹/(−n+1) + C |
| (Ax+B)/(x²+a²) | (A/2)ln(x²+a²) + (B/a)tan⁻¹(x/a) + C |
| Ax/(x²+a²) | (A/2)ln(x²+a²) + C |
| B/(x²+a²) | (B/a)tan⁻¹(x/a) + C |

---

## 3.9 Real-World Application

**Problem:** In circuit analysis, the Laplace transform inverse often requires partial fractions. For example, unfolding:

$$\frac{V(s)}{s(s+2)(s+5)} = \frac{A}{s} + \frac{B}{s+2} + \frac{C}{s+5}$$

This decomposition allows us to identify each term as a decaying exponential in the time domain, simplifying circuit response analysis.

---

## Summary Table

| Concept | Key Point |
|---|---|
| Applicability | Rational functions P(x)/Q(x) |
| Prerequisite | If deg(P) ≥ deg(Q), do long division first |
| Distinct linear | A/(x−a) for each factor |
| Repeated linear | A/(x−a) + B/(x−a)² + ··· |
| Irreducible quadratic | (Ax+B)/(ax²+bx+c) |
| Finding constants | Root substitution or equating coefficients |
| Key integrals | ln\|x−a\|, tan⁻¹(x/a), powers of (x−a) |

---

## Quick Revision Questions

1. When must you perform polynomial long division before applying partial fractions?

2. Decompose 1/[(x−1)(x+3)] into partial fractions and integrate.

3. Set up the partial fraction form for (x²+1)/[(x−2)²(x²+4)].

4. Find A, B, C in: (2x²+x+1)/[(x+1)(x²+1)] = A/(x+1) + (Bx+C)/(x²+1).

5. Evaluate ∫ dx/[x(x+1)] and simplify using logarithms.

6. Why can't you use partial fractions directly on ∫ √x/(x+1) dx?

---

[← Previous: Integration by Parts](02-integration-by-parts.md) | [Back to README](../README.md) | [Next: Trigonometric Substitutions →](04-trigonometric-substitutions.md)
