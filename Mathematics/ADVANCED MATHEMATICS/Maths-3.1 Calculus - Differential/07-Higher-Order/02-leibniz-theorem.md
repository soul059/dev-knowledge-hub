# Unit 7 — Higher-Order Derivatives: Leibniz Theorem

[← Previous: $n^{th}$ Derivatives](01-nth-derivatives.md) · [Next: Applications of Higher-Order Derivatives →](03-applications.md)

---

## Chapter Overview

**Leibniz's theorem** generalizes the product rule to the $n^{th}$ derivative of a product of two functions. It is analogous to the binomial theorem and is one of the most powerful tools for finding higher-order derivatives.

---

## 7.2.1 Successive Derivatives of a Product (Motivation)

Recall the product rule: $(uv)' = u'v + uv'$.

Now compute higher derivatives:

```
  (uv)'   = u'v + uv'                          (2 terms)
  (uv)''  = u''v + 2u'v' + uv''                (3 terms)
  (uv)''' = u'''v + 3u''v' + 3u'v'' + uv'''    (4 terms)
```

The coefficients are $1, 1$ then $1, 2, 1$ then $1, 3, 3, 1$ — **binomial coefficients!**

---

## 7.2.2 Statement of Leibniz's Theorem

If $u$ and $v$ are functions of $x$ with $n^{th}$ order derivatives, then:

$$\frac{d^n}{dx^n}(uv) = \sum_{r=0}^{n} \binom{n}{r} u^{(n-r)} v^{(r)}$$

Expanded:

$$(uv)_n = \binom{n}{0}u_n v + \binom{n}{1}u_{n-1}v_1 + \binom{n}{2}u_{n-2}v_2 + \cdots + \binom{n}{n}u \cdot v_n$$

where $\binom{n}{r} = \frac{n!}{r!(n-r)!}$ are the binomial coefficients.

```
  Compare with Binomial Theorem:

  (a + b)ⁿ = Σ C(n,r) · aⁿ⁻ʳ · bʳ
                   ↕
  (uv)ₙ   = Σ C(n,r) · u₍ₙ₋ᵣ₎ · v₍ᵣ₎

  Powers of a,b  ←→  Order of derivatives of u,v
```

---

## 7.2.3 Choosing $u$ and $v$

Choose $u$ as the function whose higher derivatives **simplify or vanish**, and $v$ as the function whose higher derivatives are **easy to find**.

| Choose as $u$ (differentiates to 0) | Choose as $v$ (easy $n^{th}$ derivative) |
|-------------------------------------|------------------------------------------|
| $x^m$ (becomes 0 after $m+1$ derivatives) | $e^{ax}$ |
| $\ln x$ | $\sin(ax+b)$, $\cos(ax+b)$ |
| $\sin^{-1}x$, $\tan^{-1}x$ | $(ax+b)^m$ |

---

## 7.2.4 Worked Examples

### Example 1: $y = x^2 e^{3x}$ — Find $y_n$

Let $u = x^2$, $v = e^{3x}$.

$$u_0 = x^2, \quad u_1 = 2x, \quad u_2 = 2, \quad u_r = 0 \text{ for } r \geq 3$$

$$v_r = 3^r e^{3x} \text{ for all } r$$

Wait — here we need $u^{(n-r)}$ and $v^{(r)}$. Let me re-assign for the terms to terminate neatly.

Let $u = x^2$ (will terminate), $v = e^{3x}$ (easy $n^{th}$ derivative).

$$(uv)_n = \sum_{r=0}^{n}\binom{n}{r}u^{(n-r)}v^{(r)}$$

But $u^{(n-r)} = 0$ when $n - r > 2$, i.e., $r < n - 2$. So only $r = n, n-1, n-2$ survive:

Actually, let me use the standard form: $(uv)_n = u_n v + n u_{n-1}v_1 + \frac{n(n-1)}{2}u_{n-2}v_2 + \cdots$

With $u = e^{3x}$ and $v = x^2$:

$$u_k = 3^k e^{3x}, \quad v_0 = x^2, \quad v_1 = 2x, \quad v_2 = 2, \quad v_k = 0 \text{ for } k \geq 3$$

$$y_n = 3^n e^{3x} \cdot x^2 + n \cdot 3^{n-1}e^{3x} \cdot 2x + \frac{n(n-1)}{2} \cdot 3^{n-2}e^{3x} \cdot 2$$

$$= e^{3x}\left[3^n x^2 + 2n \cdot 3^{n-1}x + n(n-1) \cdot 3^{n-2}\right]$$

$$= 3^{n-2}e^{3x}\left[9x^2 + 6nx + n(n-1)\right]$$

### Example 2: $y = x^3 \sin x$ — Find $y_n$

Let $u = \sin x$ (easy $n^{th}$ derivative), $v = x^3$ (terminates at $v_3$).

$$u_k = \sin\left(x + \frac{k\pi}{2}\right)$$

$$v_0 = x^3, \quad v_1 = 3x^2, \quad v_2 = 6x, \quad v_3 = 6, \quad v_k = 0 \text{ for } k \geq 4$$

$$y_n = u_n v_0 + nu_{n-1}v_1 + \frac{n(n-1)}{2}u_{n-2}v_2 + \frac{n(n-1)(n-2)}{6}u_{n-3}v_3$$

$$= x^3\sin\left(x+\frac{n\pi}{2}\right) + 3nx^2\sin\left(x+\frac{(n-1)\pi}{2}\right)$$

$$+ 3n(n-1)x\sin\left(x+\frac{(n-2)\pi}{2}\right) + n(n-1)(n-2)\sin\left(x+\frac{(n-3)\pi}{2}\right)$$

### Example 3: $y = x^2 \ln x$ — Find $y_n$ for $n \geq 3$

Let $u = x^2$, $v = \ln x$.

For $n \geq 3$: $u_k = 0$ for $k \geq 3$, and $v_k = \frac{(-1)^{k-1}(k-1)!}{x^k}$ for $k \geq 1$.

Only $r = n-2, n-1, n$ contribute (since $u^{(n-r)} = 0$ for $n - r > 2$, i.e., $r < n-2$):

$$y_n = \binom{n}{n-2}u_2 v_{n-2} + \binom{n}{n-1}u_1 v_{n-1} + \binom{n}{n}u_0 v_n$$

$$= \frac{n(n-1)}{2}\cdot 2 \cdot \frac{(-1)^{n-3}(n-3)!}{x^{n-2}} + n \cdot 2x \cdot \frac{(-1)^{n-2}(n-2)!}{x^{n-1}} + x^2 \cdot \frac{(-1)^{n-1}(n-1)!}{x^n}$$

$$= \frac{(-1)^{n-1}(n-3)!}{x^{n-2}}\left[n(n-1) - 2n(n-2) + (n-1)(n-2)\right]$$

After simplification:

$$= \frac{(-1)^{n-1}(n-3)! \cdot 2}{x^{n-2}} \quad \text{for } n \geq 3$$

Wait, let me recompute carefully. For $n \geq 3$:

$$y_n = n(n-1)\frac{(-1)^{n-3}(n-3)!}{x^{n-2}} + 2n\frac{(-1)^{n-2}(n-2)!}{x^{n-2}} + \frac{(-1)^{n-1}(n-1)!}{x^{n-2}}$$

$$= \frac{1}{x^{n-2}}\left[(-1)^{n-3}n(n-1)(n-3)! + (-1)^{n-2}\cdot 2n(n-2)! + (-1)^{n-1}(n-1)!\right]$$

$$= \frac{(-1)^{n-1}}{x^{n-2}}\left[-n(n-1)(n-3)! + 2n(n-2)! - (n-1)!\right]$$

$$= \frac{(-1)^{n-1}(n-3)!}{x^{n-2}}\left[-n(n-1) + 2n(n-2) - (n-1)(n-2)\right]$$

$$= \frac{(-1)^{n-1}(n-3)!}{x^{n-2}}\left[-n^2+n + 2n^2-4n - n^2+3n-2\right]$$

$$= \frac{(-1)^{n-1}(n-3)!}{x^{n-2}} \cdot (-2) = \frac{(-1)^n \cdot 2(n-3)!}{x^{n-2}}$$

---

## 7.2.5 Leibniz Theorem for Finding Specific Values

Sometimes we need $y_n$ evaluated at a specific point (often $x = 0$).

### Example: $y = (\sin^{-1}x)^2$, find $y_n(0)$

First, notice:

$$y_1 = 2\sin^{-1}x \cdot \frac{1}{\sqrt{1-x^2}}$$

$$y_1 \sqrt{1-x^2} = 2\sin^{-1}x$$

Differentiate again:

$$(1-x^2)y_2 - xy_1 = \frac{2}{\sqrt{1-x^2}} \cdot \sqrt{1-x^2} = 2$$

$$(1-x^2)y_2 - xy_1 = 2$$

Apply Leibniz theorem to differentiate $n$ times:

$$(1-x^2)y_{n+2} - 2nxy_{n+1} - n(n-1)y_n - xy_{n+1} - ny_n = 0$$

$$(1-x^2)y_{n+2} - (2n+1)xy_{n+1} - n^2 y_n = 0$$

At $x = 0$: $y_{n+2}(0) = n^2 y_n(0)$

With $y(0) = 0$ and $y_1(0) = 0$, $y_2(0) = 2$:

This gives a recurrence relation for computing all values.

---

## 7.2.6 Proof of Leibniz's Theorem (by Induction)

**Base case ($n = 1$):** $(uv)' = u'v + uv'$ ✓ (product rule)

**Inductive step:** Assume $(uv)_k = \sum_{r=0}^k \binom{k}{r}u_{k-r}v_r$.

Then:

$$(uv)_{k+1} = \frac{d}{dx}\left[\sum_{r=0}^k \binom{k}{r}u_{k-r}v_r\right] = \sum_{r=0}^k \binom{k}{r}[u_{k+1-r}v_r + u_{k-r}v_{r+1}]$$

Reindexing the second sum and using $\binom{k}{r} + \binom{k}{r-1} = \binom{k+1}{r}$ (Pascal's identity), we get:

$$(uv)_{k+1} = \sum_{r=0}^{k+1}\binom{k+1}{r}u_{k+1-r}v_r \quad \blacksquare$$

---

## Summary Table

| Concept | Key Formula |
|---------|-------------|
| Leibniz's theorem | $(uv)_n = \sum_{r=0}^n \binom{n}{r}u_{n-r}v_r$ |
| Binomial coefficients | $\binom{n}{r} = \frac{n!}{r!(n-r)!}$ |
| Strategy | $u$ = easy $n^{th}$ derivative, $v$ = terminates early |
| For $x^m \cdot f(x)$ | Only $m+1$ terms survive |
| Recurrence relations | Useful for $y_n(0)$ via repeated differentiation |
| Proof | By mathematical induction using Pascal's identity |

---

## Quick Revision Questions

**Q1.** State Leibniz's theorem and write the first four terms of $(uv)_n$.

**Q2.** Find the $n^{th}$ derivative of $y = x^2 e^x$ using Leibniz's theorem.

**Q3.** Find the $n^{th}$ derivative of $y = x \cos x$.

**Q4.** If $y = e^{a\sin^{-1}x}$, show that $(1-x^2)y_2 - xy_1 - a^2y = 0$.

**Q5.** Using Leibniz's theorem, find $y_5$ for $y = x^3 e^{2x}$.

**Q6.** Why is Leibniz's theorem considered analogous to the binomial theorem?

---

[← Previous: $n^{th}$ Derivatives](01-nth-derivatives.md) · [Next: Applications of Higher-Order Derivatives →](03-applications.md)
