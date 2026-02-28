# 8.2 Rouché's Theorem

[← Previous: Argument Principle](01-argument-principle.md) | [Next: Maximum Modulus Principle →](03-maximum-modulus-principle.md)

---

## Chapter Overview

**Rouché's Theorem** is a zero-counting tool: if two functions are "close" on a contour, they have the same number of zeros inside. It's the primary method for locating zeros of polynomials and analytic functions in specific regions, and it underpins stability analysis in engineering.

---

## 1. Statement

### 1.1 Classical Form

**Theorem (Rouché).** Let $f$ and $g$ be analytic inside and on a simple closed contour $C$. If

$$|f(z) - g(z)| < |f(z)| \quad \text{for all } z \in C,$$

then $f$ and $g$ have the **same number of zeros** (counted with multiplicity) inside $C$.

### 1.2 Symmetric Form

Equivalently: if

$$|f(z) - g(z)| < |f(z)| + |g(z)| \quad \text{for all } z \in C,$$

then $f$ and $g$ have the same number of zeros inside $C$.

(The symmetric form is strictly more general.)

### 1.3 "Dog Walking" Intuition

```
Think of f(z) as a person and g(z) as a dog on a leash:

    ╭───╮                Person's path: f(C)
   ╱  ○  ╲               ╭─·──╮
  │  0,0  │              ╱ ╱╲  ╲   Dog's path: g(C)
   ╲     ╱              │╱    ╲ │
    ╰───╯                ╲    ╱╱
                          ╰──·╯

If the leash (|f-g|) is always shorter than the
person's distance from the lamppost (|f|), then
person and dog wind around the lamppost the same
number of times.
```

---

## 2. Proof

**Proof.** Since $|f - g| < |f|$ on $C$, we have $f \neq 0$ on $C$. Consider:

$$h(z) = \frac{g(z)}{f(z)}$$

On $C$: $|h(z) - 1| = |g/f - 1| = |g-f|/|f| < 1$.

So $h(C)$ lies in the disk $|w - 1| < 1$, which does not contain the origin.

Therefore:

$$n(h \circ C, 0) = 0$$

By the Argument Principle:

$$0 = \frac{1}{2\pi i}\oint_C \frac{h'(z)}{h(z)}\,dz = \frac{1}{2\pi i}\oint_C \frac{g'}{g}\,dz - \frac{1}{2\pi i}\oint_C \frac{f'}{f}\,dz = Z_g - Z_f$$

Hence $Z_g = Z_f$. $\square$

---

## 3. Applications to Polynomial Zero Counting

### 3.1 Fundamental Theorem of Algebra (Quick Proof)

**Claim:** Every polynomial $p(z) = a_n z^n + \cdots + a_0$ of degree $n$ has exactly $n$ zeros in $\mathbb{C}$.

**Proof:** Let $f(z) = a_n z^n$ and $g(z) = p(z)$. On $|z| = R$ for $R$ large enough:

$$|g - f| = |a_{n-1}z^{n-1} + \cdots + a_0| \leq C R^{n-1}$$
$$|f| = |a_n| R^n$$

For $R$ sufficiently large, $|g-f| < |f|$. By Rouché, $p(z)$ has $n$ zeros inside $|z| = R$.

### 3.2 Systematic Method for Locating Zeros

**Strategy:** On a contour $C$, split $p(z) = f(z) + h(z)$ where $f$ is the dominant term and $|h| < |f|$ on $C$.

---

## 4. Design Examples

### Example 1: Zeros of $p(z) = z^7 - 5z^3 + 12$ in $|z| < 1$

**On $|z| = 1$:**
- Let $f(z) = 12$, $h(z) = z^7 - 5z^3$.
- $|h(z)| \leq |z|^7 + 5|z|^3 = 1 + 5 = 6 < 12 = |f(z)|$.

By Rouché: $p$ has the same number of zeros as $f(z) = 12$ inside $|z| < 1$, i.e., **0 zeros**.

### Example 2: Zeros of $p(z) = z^7 - 5z^3 + 12$ in $|z| < 2$

**On $|z| = 2$:**
- Let $f(z) = z^7$, $h(z) = -5z^3 + 12$.
- $|f(z)| = 2^7 = 128$.
- $|h(z)| \leq 5 \cdot 8 + 12 = 52 < 128$.

By Rouché: $p$ has the same number of zeros as $z^7$ inside $|z| < 2$, i.e., **7 zeros**.

**Conclusion:** $p$ has **0 zeros** in $|z| < 1$ and **7 zeros** in $|z| < 2$, so all **7 zeros** lie in the annulus $1 \leq |z| < 2$.

### Example 3: Zeros of $e^z = 3z$ in $|z| < 1$

Rewrite: $f(z) = e^z - 3z = 0$.

**On $|z| = 1$:**
- Let $g(z) = -3z$, $h(z) = e^z$.
- $|g(z)| = 3$ on $|z| = 1$.
- $|h(z)| = |e^z| = e^{\text{Re}(z)} \leq e^1 \approx 2.718 < 3$.

By Rouché, $e^z - 3z$ has the same number of zeros as $-3z$ in $|z| < 1$, i.e., **1 zero**.

### Example 4: How many roots of $z^4 + z^3 + 4z^2 + 2z + 3 = 0$ lie in the right half-plane?

This requires a combination of Rouché and the argument principle (or the related technique of counting sign changes on the imaginary axis). The full analysis uses indentation and substitution $z = iy$.

---

## 5. Rouché and Perturbation Theory

**Corollary.** Zeros of analytic functions depend **continuously** on parameters.

If $f_\epsilon \to f_0$ uniformly on a contour $C$ and $f_0$ has $n$ zeros inside $C$ (none on $C$), then for $\epsilon$ small enough, $f_\epsilon$ also has exactly $n$ zeros inside $C$.

This is the basis for **perturbation analysis** in applied mathematics.

---

## 6. Hurwitz's Theorem

**Theorem.** If $\{f_n\}$ converges uniformly on compact subsets to $f \not\equiv 0$, and each $f_n$ is analytic and nonzero on a domain $D$, then either $f \equiv 0$ or $f$ is nonzero on $D$.

*Proof:* Apply Rouché with $f_n$ close to $f$.

---

## 7. Summary of Zero-Counting Strategy

```
To count zeros of p(z) in a region:

  Step 1: Choose contour C bounding the region
  Step 2: Split p = f + h where f is "dominant"
  Step 3: Verify |h(z)| < |f(z)| on C
  Step 4: Count zeros of f inside C (usually obvious)
  Step 5: Conclude: p has same number of zeros

  For annuli: apply Rouché on inner and outer circles,
  subtract to get count in the annulus.
```

---

## 8. Real-World Applications

| Application | How Rouché is Used |
|-------------|-------------------|
| **Polynomial root location** | Determine how many roots lie in $|z| < R$ |
| **Control theory** | Stability = all eigenvalues in left half-plane |
| **Perturbation theory** | Small changes don't move zeros across boundaries |
| **Numerical analysis** | Validate computed eigenvalue regions |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Rouché's Theorem | $\|f-g\| < \|f\|$ on $C$ $\Rightarrow$ same zero count |
| Symmetric form | $\|f-g\| < \|f\| + \|g\|$ on $C$ suffices |
| Dominant term method | Split $p = f + h$, check $\|h\| < \|f\|$ |
| FTA proof | $z^n$ dominates lower terms for large $R$ |
| Annular location | Rouché on two circles, subtract counts |
| Hurwitz | Uniform limit inherits zero/nonzero property |
| Perturbation | Zeros move continuously with parameters |

---

## Quick Revision Questions

1. **State** Rouché's theorem and explain the "dog on a leash" analogy.

2. **Prove** the Fundamental Theorem of Algebra using Rouché's theorem.

3. **How many zeros** does $z^6 + 8z^2 + 1$ have inside $|z| < 1$? Inside $|z| < 2$?

4. **Show** that $e^z = 2z + 1$ has exactly one root in $|z| < 1$.

5. **Where** do the roots of $z^4 + z + 5$ lie? (Determine region.)

6. **State** Hurwitz's theorem and explain its relationship to Rouché's theorem.

---

[← Previous: Argument Principle](01-argument-principle.md) | [Next: Maximum Modulus Principle →](03-maximum-modulus-principle.md)
