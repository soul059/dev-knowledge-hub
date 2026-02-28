# Unit 2 — Limits: L'Hôpital's Rule

[← Previous: Indeterminate Forms](05-indeterminate-forms.md) · [Next: Unit 3 — Continuity →](../03-Continuity/01-definition-of-continuity.md)

---

## Chapter Overview

**L'Hôpital's Rule** is the most powerful tool for evaluating limits that yield indeterminate forms $\frac{0}{0}$ or $\frac{\infty}{\infty}$. It replaces the original limit with the limit of the ratio of **derivatives** of the numerator and denominator.

> **Note:** This rule uses differentiation, which is formally introduced in Units 4–5. However, it is traditionally taught alongside limits since it is a limit-evaluation tool.

---

## 2.6.1 Statement of L'Hôpital's Rule

If $\lim_{x \to a} f(x) = 0$ and $\lim_{x \to a} g(x) = 0$ (i.e., $\frac{0}{0}$ form),

**or** $\lim_{x \to a} f(x) = \pm\infty$ and $\lim_{x \to a} g(x) = \pm\infty$ (i.e., $\frac{\infty}{\infty}$ form),

and $g'(x) \ne 0$ near $a$, then:

$$\boxed{\lim_{x \to a} \frac{f(x)}{g(x)} = \lim_{x \to a} \frac{f'(x)}{g'(x)}}$$

provided the right-hand limit exists (or is $\pm\infty$).

**Also applies when $a = \pm\infty$.**

### Important Caveats

1. The rule applies **only** to $\frac{0}{0}$ or $\frac{\infty}{\infty}$. Other forms must be converted first.
2. Differentiate numerator and denominator **separately** — this is NOT the quotient rule.
3. If $\frac{f'(x)}{g'(x)}$ is still indeterminate, apply the rule **again**.
4. If $\lim \frac{f'(x)}{g'(x)}$ does not exist and is not $\pm\infty$, the rule gives no information.

---

## 2.6.2 Worked Examples: $\frac{0}{0}$ Form

### Example 1: Basic

$$\lim_{x \to 0} \frac{\sin x}{x} \quad \left(\frac{0}{0}\right)$$

$$= \lim_{x \to 0} \frac{\cos x}{1} = \frac{1}{1} = 1$$

### Example 2: Requires two applications

$$\lim_{x \to 0} \frac{1 - \cos x}{x^2} \quad \left(\frac{0}{0}\right)$$

First application:

$$= \lim_{x \to 0} \frac{\sin x}{2x} \quad \left(\text{still }\frac{0}{0}\right)$$

Second application:

$$= \lim_{x \to 0} \frac{\cos x}{2} = \frac{1}{2}$$

### Example 3: Logarithmic

$$\lim_{x \to 1} \frac{\ln x}{x - 1} \quad \left(\frac{0}{0}\right)$$

$$= \lim_{x \to 1} \frac{1/x}{1} = \frac{1}{1} = 1$$

### Example 4: Exponential vs Polynomial

$$\lim_{x \to 0} \frac{e^x - 1 - x}{x^2} \quad \left(\frac{0}{0}\right)$$

$$= \lim_{x \to 0} \frac{e^x - 1}{2x} \quad \left(\frac{0}{0}\right) = \lim_{x \to 0} \frac{e^x}{2} = \frac{1}{2}$$

---

## 2.6.3 Worked Examples: $\frac{\infty}{\infty}$ Form

### Example 1

$$\lim_{x \to \infty} \frac{\ln x}{x} \quad \left(\frac{\infty}{\infty}\right)$$

$$= \lim_{x \to \infty} \frac{1/x}{1} = \lim_{x \to \infty} \frac{1}{x} = 0$$

**Conclusion:** $\ln x$ grows **much slower** than $x$.

### Example 2

$$\lim_{x \to \infty} \frac{x^2}{e^x} \quad \left(\frac{\infty}{\infty}\right)$$

$$= \lim_{x \to \infty} \frac{2x}{e^x} \quad \left(\frac{\infty}{\infty}\right) = \lim_{x \to \infty} \frac{2}{e^x} = 0$$

**Conclusion:** $e^x$ grows **much faster** than any polynomial.

### Growth rate hierarchy

$$\ln x \ll x^n \ll e^x \ll x^x \quad \text{as } x \to \infty$$

---

## 2.6.4 Other Indeterminate Forms via L'Hôpital

### Form: $0 \cdot \infty$

**Strategy:** Rewrite as $\frac{0}{0}$ or $\frac{\infty}{\infty}$.

$$\lim_{x \to 0^+} x \ln x \quad (0 \cdot (-\infty))$$

Rewrite: $= \lim_{x \to 0^+} \frac{\ln x}{1/x} \quad \left(\frac{-\infty}{\infty}\right)$

Apply L'Hôpital:

$$= \lim_{x \to 0^+} \frac{1/x}{-1/x^2} = \lim_{x \to 0^+} \frac{x^2}{-x} = \lim_{x \to 0^+} (-x) = 0$$

### Form: $\infty - \infty$

$$\lim_{x \to 0} \left(\frac{1}{x} - \frac{1}{\sin x}\right) \quad (\infty - \infty)$$

Combine: $= \lim_{x \to 0} \frac{\sin x - x}{x \sin x} \quad \left(\frac{0}{0}\right)$

$$= \lim_{x \to 0} \frac{\cos x - 1}{\sin x + x\cos x} \quad \left(\frac{0}{0}\right) = \lim_{x \to 0} \frac{-\sin x}{\cos x + \cos x - x\sin x} = \frac{0}{2} = 0$$

### Form: $0^0$, $1^\infty$, $\infty^0$ — The Logarithm Trick

For $\lim f(x)^{g(x)}$:

1. Let $y = f(x)^{g(x)}$
2. $\ln y = g(x) \cdot \ln f(x)$
3. Evaluate $\lim \ln y$ (usually $0 \cdot \infty$ → convert)
4. Then $\lim y = e^{\lim \ln y}$

**Example ($0^0$):**

$$\lim_{x \to 0^+} x^x$$

$\ln y = x \ln x \to 0$ (shown above). So $\lim y = e^0 = 1$.

**Example ($\infty^0$):**

$$\lim_{x \to \infty} x^{1/x}$$

$\ln y = \frac{\ln x}{x} \to 0$ (shown above). So $\lim y = e^0 = 1$.

---

## 2.6.5 Common Mistakes

| Mistake | Why It's Wrong |
|---------|---------------|
| Applying L'Hôpital to non-indeterminate forms | The rule only applies to $\frac{0}{0}$ or $\frac{\infty}{\infty}$ |
| Using the quotient rule instead | We differentiate numerator and denominator **separately** |
| Forgetting to check the form | Always verify indeterminate form **before** applying |
| Infinite loop of L'Hôpital | Some limits need algebraic manipulation instead (e.g., $\frac{e^x + e^{-x}}{e^x - e^{-x}}$ as $x \to \infty$) |

---

## 2.6.6 Process Flowchart

```
   Is the limit  0/0  or  ∞/∞ ?
          │
     ┌────┴────┐
    YES        NO
     │          │
  Apply        Is it  0·∞, ∞-∞, 0⁰, 1^∞, ∞⁰ ?
  L'Hôpital         │
     │          ┌────┴────┐
     │        YES         NO
     │         │           │
     │    Convert to     Regular
     │    0/0 or ∞/∞     evaluation
     │         │
     ▼         ▼
   f'(x)/g'(x)
     │
   Still indeterminate?
     │
   ┌─┴──┐
  YES    NO → Answer found!
   │
  Apply again
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Applies to | $\frac{0}{0}$ and $\frac{\infty}{\infty}$ forms only |
| Formula | $\lim \frac{f}{g} = \lim \frac{f'}{g'}$ |
| Can repeat | Yes, if result is still indeterminate |
| Other forms | Convert to $\frac{0}{0}$ or $\frac{\infty}{\infty}$ first |
| Exponential forms | Use $\ln y = g \cdot \ln f$, then $y = e^{\lim \ln y}$ |
| Not the quotient rule | Differentiate numerator and denominator **separately** |
| May fail | If derivative ratio has no limit and isn't $\pm\infty$ |

---

## Quick Revision Questions

**Q1.** State L'Hôpital's Rule precisely. Under what conditions does it apply?

**Q2.** Evaluate $\lim_{x \to 0} \frac{e^x - 1}{x}$ using L'Hôpital's Rule.

**Q3.** Find $\lim_{x \to \infty} \frac{x^3}{e^x}$. How many times must you apply L'Hôpital?

**Q4.** Evaluate $\lim_{x \to 0^+} x^{\sin x}$ (identify the indeterminate form first).

**Q5.** Why can't L'Hôpital's Rule be applied to $\lim_{x \to 0} \frac{\sin x + 1}{x + 1}$?

**Q6.** Use L'Hôpital's Rule to find $\lim_{x \to 0} \frac{x - \sin x}{x^3}$.

---

[← Previous: Indeterminate Forms](05-indeterminate-forms.md) · [Next: Unit 3 — Continuity →](../03-Continuity/01-definition-of-continuity.md)
