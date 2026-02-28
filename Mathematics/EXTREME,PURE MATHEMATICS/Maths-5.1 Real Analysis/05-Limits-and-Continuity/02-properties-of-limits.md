# Chapter 2: Properties of Limits

[← Previous: Limit of a Function](01-limit-of-a-function.md) | [Back to README](../README.md) | [Next: Continuity — Definition →](03-continuity-definition.md)

---

## Overview

Just as with sequences, function limits obey **algebraic rules** (sum, product, quotient) and **order rules** (squeeze theorem, sign preservation). These theorems let us compute limits of complex functions from simpler ones without revisiting $\varepsilon$-$\delta$ each time.

---

## 2.1 Algebraic Limit Theorems

**Theorem 2.1.** Suppose $\lim_{x \to c} f(x) = L$ and $\lim_{x \to c} g(x) = M$. Then:

| Rule | Statement |
|------|-----------|
| **Sum** | $\lim_{x \to c}[f(x) + g(x)] = L + M$ |
| **Scalar** | $\lim_{x \to c}[k \cdot f(x)] = kL$ |
| **Product** | $\lim_{x \to c}[f(x) \cdot g(x)] = L \cdot M$ |
| **Quotient** | $\lim_{x \to c}\frac{f(x)}{g(x)} = \frac{L}{M}$ (if $M \neq 0$) |
| **Power** | $\lim_{x \to c}[f(x)]^n = L^n$ |

**Proof strategy:** Can be proved directly ($\varepsilon$-$\delta$) or via the Sequential Criterion + algebraic limit theorems for sequences.

**Proof of Product (via Sequential Criterion).** Let $(x_n) \to c$ with $x_n \neq c$. Then $f(x_n) \to L$ and $g(x_n) \to M$, so $f(x_n)g(x_n) \to LM$ by ALT for sequences. Since this holds for every such sequence, $\lim_{x \to c} f(x)g(x) = LM$. $\blacksquare$

---

## 2.2 Order Properties

**Theorem 2.2 (Sign Preservation).** If $\lim_{x \to c} f(x) = L > 0$, then $\exists\, \delta > 0$ such that $f(x) > 0$ for $0 < |x - c| < \delta$.

**Theorem 2.3 (Order Limit).** If $\lim_{x \to c} f(x) = L$, $\lim_{x \to c} g(x) = M$, and $f(x) \leq g(x)$ near $c$, then $L \leq M$.

**Theorem 2.4 (Squeeze/Sandwich Theorem).** If $f(x) \leq h(x) \leq g(x)$ near $c$, and $\lim_{x \to c} f(x) = \lim_{x \to c} g(x) = L$, then $\lim_{x \to c} h(x) = L$.

```
  Squeeze Theorem:

  y ▲
    │    g(x)
    │   ╱  ╲
  L │─╱─●───╲──── h(x) squeezed between
    │╱   ╱╲  ╲
    │  ╱    ╲
    │ f(x)
    └──────┼──────► x
           c
  
  f(x) ≤ h(x) ≤ g(x)  and  f,g → L  ⟹  h → L
```

---

## 2.3 Important Limits

| Limit | Value | Method |
|-------|:-----:|--------|
| $\lim_{x \to 0} \frac{\sin x}{x}$ | $1$ | Squeeze: $\cos x \leq \frac{\sin x}{x} \leq 1$ |
| $\lim_{x \to 0} \frac{1 - \cos x}{x}$ | $0$ | Use $1 - \cos x = 2\sin^2(x/2)$ |
| $\lim_{x \to 0} \frac{1 - \cos x}{x^2}$ | $1/2$ | Same identity |
| $\lim_{x \to 0} \frac{e^x - 1}{x}$ | $1$ | Definition of $e^x$ derivative |
| $\lim_{x \to 0} \frac{\ln(1+x)}{x}$ | $1$ | Substitution from above |
| $\lim_{x \to \infty} (1 + 1/x)^x$ | $e$ | Definition of $e$ |
| $\lim_{x \to 0^+} x \ln x$ | $0$ | L'Hôpital or substitution |

---

## 2.4 Composition of Limits

**Theorem 2.5.** If $\lim_{x \to c} g(x) = L$ and $f$ is continuous at $L$, then:

$$\lim_{x \to c} f(g(x)) = f(L)$$

**Caution:** Without continuity of $f$, composition may fail. We also need $g(x) \neq L$ near $c$, or $f$ to be defined at $L$.

---

## 2.5 Limits and Absolute Value

**Theorem 2.6.** $\lim_{x \to c} f(x) = 0 \;\iff\; \lim_{x \to c} |f(x)| = 0$.

**Warning:** $\lim_{x \to c} |f(x)| = L$ does **not** imply $\lim_{x \to c} f(x) = L$ (or even exists) unless $L = 0$. Example: $f(x) = (-1)^{\lfloor 1/x \rfloor}$ near $0$ has $|f(x)| = 1$ but $f(x)$ oscillates.

---

## Summary Table

| Property | Statement |
|----------|-----------|
| Sum rule | $\lim(f + g) = \lim f + \lim g$ |
| Product rule | $\lim(fg) = (\lim f)(\lim g)$ |
| Quotient rule | $\lim(f/g) = (\lim f)/(\lim g)$ if $\lim g \neq 0$ |
| Squeeze | $f \leq h \leq g$ and $\lim f = \lim g = L$ $\Rightarrow$ $\lim h = L$ |
| Order limit | $f \leq g$ near $c$ $\Rightarrow$ $\lim f \leq \lim g$ |
| Composition | $\lim f(g(x)) = f(\lim g(x))$ if $f$ continuous |
| Sequential proof strategy | Use Sequential Criterion + sequence ALT |
| $\sin x / x \to 1$ | Fundamental limit for calculus |

---

## Quick Revision Questions

1. **State** the Algebraic Limit Theorems for functions.

2. **Prove** the product rule for limits using the Sequential Criterion.

3. **Use** the Squeeze Theorem to prove $\lim_{x \to 0} x^2 \sin(1/x) = 0$.

4. **Prove** $\lim_{x \to 0} \frac{\sin x}{x} = 1$ using the squeeze inequality.

5. **Why** can't we always compose limits? Give a counterexample.

6. **True or False:** If $\lim_{x \to c} |f(x)| = 5$, then $\lim_{x \to c} f(x) = 5$. Justify.

---

[← Previous: Limit of a Function](01-limit-of-a-function.md) | [Back to README](../README.md) | [Next: Continuity — Definition →](03-continuity-definition.md)
