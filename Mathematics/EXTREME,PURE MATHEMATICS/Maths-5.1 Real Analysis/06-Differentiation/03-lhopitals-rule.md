# Chapter 3: L'Hôpital's Rule

[← Previous: Mean Value Theorems](02-mean-value-theorems.md) | [Back to README](../README.md) | [Next: Taylor's Theorem →](04-taylors-theorem.md)

---

## Overview

**L'Hôpital's Rule** evaluates indeterminate forms $0/0$ and $\infty/\infty$ by differentiating numerator and denominator. Its rigorous proof uses the **Cauchy Mean Value Theorem**.

---

## 3.1 The 0/0 Form

**Theorem 3.1 (L'Hôpital — $0/0$).** Let $f, g$ be differentiable on $(a, b)$ with $g'(x) \neq 0$. If:
- $\lim_{x \to a^+} f(x) = 0$ and $\lim_{x \to a^+} g(x) = 0$
- $\lim_{x \to a^+} \frac{f'(x)}{g'(x)} = L$ (finite, $+\infty$, or $-\infty$)

Then $\lim_{x \to a^+} \frac{f(x)}{g(x)} = L$.

**Proof.** Define $f(a) = g(a) = 0$ to make $f, g$ continuous on $[a, b)$. For $x \in (a, b)$, by Cauchy MVT, $\exists\, c_x \in (a, x)$ with:

$$\frac{f(x)}{g(x)} = \frac{f(x) - f(a)}{g(x) - g(a)} = \frac{f'(c_x)}{g'(c_x)}$$

As $x \to a^+$, $c_x \to a^+$ (since $a < c_x < x$), so:

$$\lim_{x \to a^+} \frac{f(x)}{g(x)} = \lim_{c_x \to a^+} \frac{f'(c_x)}{g'(c_x)} = L. \quad\blacksquare$$

---

## 3.2 The ∞/∞ Form

**Theorem 3.2 (L'Hôpital — $\infty/\infty$).** Same hypotheses except:
- $\lim_{x \to a^+} |f(x)| = \infty$ and $\lim_{x \to a^+} |g(x)| = \infty$

Then $\lim_{x \to a^+} \frac{f(x)}{g(x)} = L$ if $\lim_{x \to a^+} \frac{f'(x)}{g'(x)} = L$.

(The proof is more technical; uses Cauchy MVT on $[x, y]$ and careful limit arguments.)

---

## 3.3 Other Indeterminate Forms

All can be converted to $0/0$ or $\infty/\infty$:

| Form | Conversion |
|------|-----------|
| $0 \cdot \infty$ | $f \cdot g = f/(1/g)$ → $0/0$ or $\infty/\infty$ |
| $\infty - \infty$ | Combine into single fraction |
| $1^\infty$ | $f^g = e^{g \ln f}$; compute $\lim g \ln f$ |
| $0^0$ | $f^g = e^{g \ln f}$; compute $\lim g \ln f$ (gives $0 \cdot (-\infty)$) |
| $\infty^0$ | $f^g = e^{g \ln f}$; compute $\lim g \ln f$ |

---

## 3.4 Worked Examples

**Example 1.** $\lim_{x \to 0} \frac{\sin x}{x}$ ($0/0$):

$$\lim_{x \to 0} \frac{\sin x}{x} = \lim_{x \to 0} \frac{\cos x}{1} = 1$$

**Example 2.** $\lim_{x \to \infty} \frac{x}{e^x}$ ($\infty/\infty$):

$$\lim_{x \to \infty}\frac{x}{e^x} = \lim_{x \to \infty} \frac{1}{e^x} = 0$$

**Example 3.** $\lim_{x \to 0^+} x \ln x$ ($0 \cdot (-\infty)$):

$$= \lim_{x \to 0^+} \frac{\ln x}{1/x} = \lim_{x \to 0^+} \frac{1/x}{-1/x^2} = \lim_{x \to 0^+} (-x) = 0$$

**Example 4.** $\lim_{x \to 0^+} x^x$ ($0^0$):

$$x^x = e^{x\ln x} \to e^0 = 1 \quad\text{since } x \ln x \to 0$$

---

## 3.5 Common Pitfalls

**Pitfall 1: Applying when not indeterminate.**
$$\frac{\sin x}{\cos x} \;\text{at}\; x = 0 \quad\text{is}\quad \frac{0}{1} = 0 \quad\text{(not indeterminate!)}$$

**Pitfall 2: Circular reasoning.** Using $\lim \frac{\sin x}{x} = 1$ to prove $(\sin x)' = \cos x$, then L'Hôpital to prove $\lim \frac{\sin x}{x} = 1$.

**Pitfall 3: Limit of $f'/g'$ doesn't exist.** L'Hôpital requires $\lim f'/g'$ to exist. If it doesn't, the rule says nothing.

**Example:** $\lim_{x \to \infty} \frac{x + \sin x}{x}$. L'Hôpital gives $\frac{1 + \cos x}{1}$, which doesn't converge. But the original limit is $1$ (divide by $x$).

```
  ⚠️ L'Hôpital only works when:
  1. The form IS indeterminate (0/0 or ∞/∞)
  2. The limit of f'/g' EXISTS (or is ±∞)
  3. g'(x) ≠ 0 near the point
  
  If ANY of these fail, L'Hôpital cannot be applied!
```

---

## Summary Table

| Concept | Key Result |
|---------|------------|
| L'Hôpital ($0/0$) | $\lim f/g = \lim f'/g'$ if $0/0$ form |
| L'Hôpital ($\infty/\infty$) | Same conclusion for $\infty/\infty$ |
| Proof tool | Cauchy Mean Value Theorem |
| Other forms | Convert $0 \cdot \infty$, $1^\infty$, etc. to $0/0$ or $\infty/\infty$ |
| Pitfall | Must verify indeterminate form and existence of $\lim f'/g'$ |

---

## Quick Revision Questions

1. **Prove** L'Hôpital's rule for the $0/0$ case using Cauchy MVT.

2. **Evaluate** $\lim_{x \to 0}\frac{e^x - 1 - x}{x^2}$.

3. **Evaluate** $\lim_{x \to \infty} x^{1/x}$.

4. **Give** an example where L'Hôpital is applicable repeatedly (3+ times).

5. **Show** an example where $\lim f/g$ exists but $\lim f'/g'$ does not.

6. **Why** is L'Hôpital's proof for the $\infty/\infty$ case more difficult?

---

[← Previous: Mean Value Theorems](02-mean-value-theorems.md) | [Back to README](../README.md) | [Next: Taylor's Theorem →](04-taylors-theorem.md)
