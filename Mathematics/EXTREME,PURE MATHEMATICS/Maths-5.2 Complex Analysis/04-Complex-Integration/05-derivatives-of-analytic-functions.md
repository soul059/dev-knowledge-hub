# 4.5 Derivatives of Analytic Functions

[← Previous: Cauchy Integral Formula](04-cauchy-integral-formula.md) | [Next: Morera's Theorem →](06-moreras-theorem.md)

---

## Chapter Overview

We have seen that the Cauchy Integral Formula implies every analytic function is infinitely differentiable. This chapter explores the deeper consequences: **Liouville's theorem**, the **Fundamental Theorem of Algebra**, and the remarkable rigidity of analytic functions.

---

## 1. Infinite Differentiability

### 1.1 The Main Result

> **Theorem.** If $f$ is analytic in a domain $D$, then $f', f'', f''', \ldots$ all exist and are analytic in $D$.

**Proof:** From the generalized CIF with any circle $C$ around $z_0 \in D$:

$$f^{(n)}(z_0) = \frac{n!}{2\pi i}\oint_C \frac{f(z)}{(z - z_0)^{n+1}}\,dz$$

The right-hand side exists for all $n \geq 0$, and defines an analytic function of $z_0$. $\square$

### 1.2 Comparison with Real Analysis

```
Real Analysis:        Complex Analysis:

  f differentiable           f differentiable (analytic)
       │                              │
       ↓                              ↓
  f' may or may            f' exists AND is
  not be continuous          analytic
       │                              │
       ↓                              ↓
  f'' may not              f'' exists AND is
  exist at all               analytic
       │                              │
       ↓                              ↓
     ...                   f is C^∞ (infinitely
                           differentiable)
                                      │
                                      ↓
                           f has convergent
                           Taylor series!
```

---

## 2. Liouville's Theorem

### 2.1 Statement

> **Liouville's Theorem.** If $f$ is **entire** (analytic on all of $\mathbb{C}$) and **bounded** (there exists $M$ with $|f(z)| \leq M$ for all $z$), then $f$ is **constant**.

### 2.2 Proof

By Cauchy's inequality with $n = 1$, for any $z_0$ and any $R > 0$:

$$|f'(z_0)| \leq \frac{M}{R}$$

Let $R \to \infty$: $|f'(z_0)| = 0$. Since $z_0$ is arbitrary, $f' \equiv 0$, so $f$ is constant. $\square$

### 2.3 Generalization

If $f$ is entire and $|f(z)| \leq C|z|^k$ for large $|z|$, then $f$ is a polynomial of degree $\leq k$.

**Proof:** Cauchy's inequality gives $|f^{(k+1)}(z_0)| \leq \frac{(k+1)!\,C R^k}{R^{k+1}} \to 0$, so $f^{(k+1)} \equiv 0$.

---

## 3. The Fundamental Theorem of Algebra

### 3.1 Statement

Every non-constant polynomial $p(z) = a_nz^n + \cdots + a_0$ with $a_n \neq 0$ and $n \geq 1$ has at least one root in $\mathbb{C}$.

### 3.2 Proof via Liouville's Theorem

**By contradiction:** Suppose $p(z) \neq 0$ for all $z \in \mathbb{C}$.

**Step 1.** Then $g(z) = 1/p(z)$ is entire.

**Step 2.** For $|z|$ large, $|p(z)| \approx |a_n||z|^n \to \infty$, so $|g(z)| \to 0$.

**Step 3.** $g$ is continuous on the compact set $|z| \leq R$, hence bounded there.

**Step 4.** For $|z| > R$ (large enough), $|g(z)| < 1$.

**Step 5.** Therefore $g$ is bounded on all of $\mathbb{C}$.

**Step 6.** By Liouville's theorem, $g$ is constant, hence $p$ is constant — contradiction! $\square$

### 3.3 Corollary

Every polynomial of degree $n$ has exactly $n$ roots (counted with multiplicity) in $\mathbb{C}$.

---

## 4. Further Consequences

### 4.1 Taylor's Theorem for Analytic Functions

If $f$ is analytic in $|z - z_0| < R$, then:

$$f(z) = \sum_{n=0}^{\infty} \frac{f^{(n)}(z_0)}{n!}(z - z_0)^n$$

converging for $|z - z_0| < R$. (Full treatment in Unit 5.)

### 4.2 Morera's Converse (Preview)

If $f$ is continuous on $D$ and $\oint_\gamma f\,dz = 0$ for every closed contour in $D$, then $f$ is analytic (and hence infinitely differentiable). See next chapter.

---

## 5. Design Example

### Example: Show that if $f$ is entire and $\text{Re}(f(z)) \leq 100$ for all $z$, then $f$ is constant.

**Step 1.** Consider $g(z) = e^{f(z)}$. Then $g$ is entire.

**Step 2.** $|g(z)| = |e^{f(z)}| = e^{\text{Re}(f(z))} \leq e^{100}$.

**Step 3.** So $g$ is entire and bounded.

**Step 4.** By Liouville's theorem, $g$ is constant.

**Step 5.** $e^{f(z)} = C$ implies $f(z) = \log C + 2\pi i k$ for some integer $k$. By continuity, $f$ is constant. $\square$

### Example 2: Compute $f^{(4)}(0)$ where $f(z) = \frac{z}{z^2+1}$

By the generalized CIF with $C: |z| = 1/2$:

$$f^{(4)}(0) = \frac{4!}{2\pi i}\oint_{|z|=1/2} \frac{f(z)}{z^5}\,dz = \frac{24}{2\pi i}\oint_{|z|=1/2} \frac{1}{z^4(z^2+1)}\,dz$$

Alternatively, use Taylor expansion:

$$\frac{z}{z^2+1} = z \cdot \frac{1}{1+z^2} = z(1 - z^2 + z^4 - \cdots) = z - z^3 + z^5 - \cdots$$

Coefficient of $z^4$ is $0$, so $\frac{f^{(4)}(0)}{4!} = 0$, giving $f^{(4)}(0) = 0$.

---

## 6. Real-World Applications

| Application | Theorem Used |
|-------------|-------------|
| **Quantum mechanics** | Entire wavefunctions that are bounded must be constant → quantization |
| **Signal processing** | Bounded analytic signals have strong rigidity |
| **Algebra** | FTA guarantees factorization of characteristic polynomials |
| **Control theory** | Polynomial equations for transfer function poles always have solutions in $\mathbb{C}$ |

---

## Summary Table

| Result | Statement |
|--------|-----------|
| Infinite differentiability | Analytic $\Rightarrow f^{(n)}$ exists and is analytic for all $n$ |
| Cauchy's inequality | $\|f^{(n)}(z_0)\| \leq n!\,M(R)/R^n$ |
| Liouville's theorem | Entire + bounded $\Rightarrow$ constant |
| Generalized Liouville | Entire + $O(\|z\|^k)$ $\Rightarrow$ polynomial of degree $\leq k$ |
| Fundamental Theorem of Algebra | Every non-constant polynomial has a root in $\mathbb{C}$ |
| Bounded real part | Entire + $\text{Re}(f)$ bounded $\Rightarrow$ constant |

---

## Quick Revision Questions

1. **Prove** Liouville's theorem using Cauchy's inequality.

2. **Prove** the Fundamental Theorem of Algebra using Liouville's theorem.

3. **Show** that if $f$ is entire and $|f(z)| \leq 3|z|^2 + 7$ for all $z$, then $f$ is a polynomial of degree at most $2$.

4. **Explain** why Liouville's theorem fails for real-valued functions on $\mathbb{R}$ (give a counterexample).

5. **Find** $f^{(5)}(0)$ where $f(z) = \frac{1}{1+z^2}$ using the Taylor series approach.

6. **Prove** that an entire function with $|f(z)| \geq 1$ for all $z$ is constant.

---

[← Previous: Cauchy Integral Formula](04-cauchy-integral-formula.md) | [Next: Morera's Theorem →](06-moreras-theorem.md)
