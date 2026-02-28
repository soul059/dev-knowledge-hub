# Unit 2 — Limits: Laws of Limits

[← Previous: Left and Right Hand Limits](02-left-right-limits.md) · [Next: Evaluation Techniques →](04-evaluation-techniques.md)

---

## Chapter Overview

The **laws of limits** allow us to break apart complex limit expressions into simpler pieces. Instead of evaluating every limit from the ε-δ definition, we use these algebraic rules to compute limits efficiently.

---

## 2.3.1 Basic Limit Laws

Suppose $\lim_{x \to a} f(x) = L$ and $\lim_{x \to a} g(x) = M$, where $L$ and $M$ are finite real numbers. Then:

| # | Law | Statement |
|---|-----|-----------|
| 1 | **Constant** | $\lim_{x \to a} c = c$ |
| 2 | **Identity** | $\lim_{x \to a} x = a$ |
| 3 | **Sum** | $\lim_{x \to a} [f(x) + g(x)] = L + M$ |
| 4 | **Difference** | $\lim_{x \to a} [f(x) - g(x)] = L - M$ |
| 5 | **Constant Multiple** | $\lim_{x \to a} [c \cdot f(x)] = c \cdot L$ |
| 6 | **Product** | $\lim_{x \to a} [f(x) \cdot g(x)] = L \cdot M$ |
| 7 | **Quotient** | $\lim_{x \to a} \frac{f(x)}{g(x)} = \frac{L}{M}$, provided $M \ne 0$ |
| 8 | **Power** | $\lim_{x \to a} [f(x)]^n = L^n$ (for positive integer $n$) |
| 9 | **Root** | $\lim_{x \to a} \sqrt[n]{f(x)} = \sqrt[n]{L}$ (provided $L \ge 0$ for even $n$) |

---

## 2.3.2 Applying the Laws — Worked Examples

### Example 1: Direct application

$$\lim_{x \to 2} (3x^2 - 5x + 4)$$

Using sum, constant multiple, and power laws:

$$= 3 \cdot \lim_{x \to 2} x^2 - 5 \cdot \lim_{x \to 2} x + \lim_{x \to 2} 4$$
$$= 3(4) - 5(2) + 4 = 12 - 10 + 4 = 6$$

### Example 2: Quotient law

$$\lim_{x \to 3} \frac{x^2 + 1}{x - 1} = \frac{\lim_{x \to 3}(x^2 + 1)}{\lim_{x \to 3}(x - 1)} = \frac{9 + 1}{3 - 1} = \frac{10}{2} = 5$$

This works because the denominator limit is $2 \ne 0$.

### Example 3: When direct substitution fails

$$\lim_{x \to 1} \frac{x^2 - 1}{x - 1}$$

Direct substitution gives $\frac{0}{0}$. The quotient law **does not apply** when $M = 0$. We must simplify first (see Evaluation Techniques).

---

## 2.3.3 Direct Substitution Property

For any **polynomial** or **rational function** (with nonzero denominator at $a$):

$$\lim_{x \to a} p(x) = p(a) \qquad \text{(polynomials)}$$

$$\lim_{x \to a} \frac{p(x)}{q(x)} = \frac{p(a)}{q(a)} \qquad \text{(rational, } q(a) \ne 0\text{)}$$

Also applies to:
- Trigonometric functions at points in their domain
- Exponential and logarithmic functions at points in their domain
- Compositions of the above

---

## 2.3.4 The Squeeze Theorem (Sandwich Theorem)

### Statement

If $g(x) \le f(x) \le h(x)$ for all $x$ near $a$ (except possibly at $a$), and:

$$\lim_{x \to a} g(x) = \lim_{x \to a} h(x) = L$$

then:

$$\lim_{x \to a} f(x) = L$$

### Visualisation

```
  y
  │
  │    h(x) ╲     ╱ h(x)
  │          ╲   ╱
  L├─ ─ ─ ─ ─╲─╱─ ─ ─ ─ ─ ─ ─   ← all three converge to L
  │          ╱ ╲
  │    g(x) ╱   ╲ g(x)
  │        ╱     ╲
  │       ╱ f(x)  ╲
  │      ╱ squeezed ╲
  └──────────┼──────────── x
             a
```

$f$ is "squeezed" between $g$ and $h$, both of which approach $L$.

### Classic Application

**Prove:** $\lim_{x \to 0} x^2 \sin\left(\frac{1}{x}\right) = 0$

We know $-1 \le \sin\left(\frac{1}{x}\right) \le 1$, so:

$$-x^2 \le x^2 \sin\left(\frac{1}{x}\right) \le x^2$$

Since $\lim_{x \to 0} (-x^2) = 0$ and $\lim_{x \to 0} x^2 = 0$:

$$\lim_{x \to 0} x^2 \sin\left(\frac{1}{x}\right) = 0 \quad \text{(by Squeeze Theorem)}$$

---

## 2.3.5 Important Standard Limits

These results are used extensively and should be memorized:

| # | Standard Limit | Value |
|---|---------------|-------|
| 1 | $\lim_{x \to 0} \frac{\sin x}{x}$ | $1$ |
| 2 | $\lim_{x \to 0} \frac{\tan x}{x}$ | $1$ |
| 3 | $\lim_{x \to 0} \frac{1 - \cos x}{x^2}$ | $\frac{1}{2}$ |
| 4 | $\lim_{x \to 0} \frac{e^x - 1}{x}$ | $1$ |
| 5 | $\lim_{x \to 0} \frac{a^x - 1}{x}$ | $\ln a$ |
| 6 | $\lim_{x \to 0} \frac{\ln(1 + x)}{x}$ | $1$ |
| 7 | $\lim_{x \to 0} (1 + x)^{1/x}$ | $e$ |
| 8 | $\lim_{x \to \infty} \left(1 + \frac{1}{x}\right)^x$ | $e$ |
| 9 | $\lim_{x \to 0} \frac{\sin^{-1} x}{x}$ | $1$ |
| 10 | $\lim_{x \to 0} \frac{\tan^{-1} x}{x}$ | $1$ |

### Proof sketch for $\lim_{x \to 0} \frac{\sin x}{x} = 1$

Using the unit circle and Squeeze Theorem:

For $0 < x < \frac{\pi}{2}$:

$$\sin x < x < \tan x$$

Dividing by $\sin x > 0$:

$$1 < \frac{x}{\sin x} < \frac{1}{\cos x}$$

Taking reciprocals (reversing inequalities):

$$\cos x < \frac{\sin x}{x} < 1$$

As $x \to 0^+$, $\cos x \to 1$ and $1 \to 1$, so by Squeeze Theorem:

$$\lim_{x \to 0^+} \frac{\sin x}{x} = 1$$

Since $\frac{\sin x}{x}$ is an even function, the left-hand limit is also 1.

---

## 2.3.6 Laws for Limits at Infinity

All the basic laws (sum, product, quotient, etc.) also hold for limits as $x \to \infty$ or $x \to -\infty$.

**Key result:**

$$\lim_{x \to \infty} \frac{1}{x^n} = 0 \quad \text{for all } n > 0$$

**Strategy for rational functions at infinity:** Divide numerator and denominator by the highest power of $x$ in the denominator.

**Example:**

$$\lim_{x \to \infty} \frac{3x^2 + 2x - 1}{5x^2 - x + 4}$$

Divide by $x^2$:

$$= \lim_{x \to \infty} \frac{3 + \frac{2}{x} - \frac{1}{x^2}}{5 - \frac{1}{x} + \frac{4}{x^2}} = \frac{3 + 0 - 0}{5 - 0 + 0} = \frac{3}{5}$$

**General rule for $\frac{a_n x^n + \cdots}{b_m x^m + \cdots}$:**

| Degree comparison | Limit as $x \to \infty$ |
|-------------------|------------------------|
| $n < m$ | $0$ |
| $n = m$ | $\frac{a_n}{b_m}$ (ratio of leading coefficients) |
| $n > m$ | $\pm \infty$ |

---

## Summary Table

| Law / Theorem | When to Use |
|---------------|-------------|
| Sum / Difference | Break apart $\lim[f \pm g]$ |
| Product | Evaluate $\lim[f \cdot g]$ |
| Quotient | Evaluate $\lim[f/g]$, only if denom limit ≠ 0 |
| Direct substitution | Polynomials, rationals (denom ≠ 0), standard functions |
| Squeeze Theorem | $f$ trapped between two functions with same limit |
| Standard limits | Trig, exponential limits near 0 |
| Limits at $\infty$ | Divide by highest power in denominator |

---

## Quick Revision Questions

**Q1.** State the five basic limit laws (sum, difference, product, quotient, power).

**Q2.** Evaluate $\lim_{x \to -1} (x^3 + 3x^2 - x + 5)$ using direct substitution.

**Q3.** Use the Squeeze Theorem to prove $\lim_{x \to 0} x \cos\left(\frac{1}{x}\right) = 0$.

**Q4.** What is $\lim_{x \to \infty} \frac{2x^3 + x}{7x^3 - 3x^2}$?

**Q5.** Why can't we apply the quotient law directly to $\lim_{x \to 2} \frac{x^2 - 4}{x - 2}$?

**Q6.** Write three standard limits involving trigonometric functions that evaluate to 1.

---

[← Previous: Left and Right Hand Limits](02-left-right-limits.md) · [Next: Evaluation Techniques →](04-evaluation-techniques.md)
