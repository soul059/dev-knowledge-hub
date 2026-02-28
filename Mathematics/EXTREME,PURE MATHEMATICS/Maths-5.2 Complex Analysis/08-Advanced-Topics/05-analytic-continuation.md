# 8.5 Analytic Continuation

[← Previous: Schwarz Lemma](04-schwarz-lemma.md) | [Back to README →](../README.md)

---

## Chapter Overview

**Analytic continuation** is the process of extending the domain of an analytic function beyond its original region of definition while preserving analyticity. It reveals that many "different" formulas define the same underlying analytic object and is the mechanism behind the Riemann zeta function's extension to the entire complex plane.

---

## 1. The Idea

A function $f$ defined on domain $D_1$ and a function $g$ defined on domain $D_2$ are **analytic continuations** of each other if:

1. $D_1 \cap D_2 \neq \emptyset$
2. $f = g$ on $D_1 \cap D_2$

The combined function $F$ on $D_1 \cup D_2$ is the analytic continuation.

```
         D₁                  D₂
    ╭─────────╮         ╭─────────╮
   ╱           ╲       ╱           ╲
  │    f(z)     ├─────┤    g(z)     │
  │             │ f=g │             │
   ╲           ╱       ╲           ╱
    ╰─────────╯         ╰─────────╯
              overlap
```

---

## 2. Uniqueness: The Identity Theorem Revisited

**Theorem.** If $f$ and $g$ are analytic on a domain $D$ and $f = g$ on a set with a limit point in $D$, then $f = g$ on all of $D$.

**Corollary.** Analytic continuation, when it exists, is **unique**.

This means: there is only **one** way to extend an analytic function along a given path.

---

## 3. Continuation Along a Path

### 3.1 Chain of Disks

To continue $f$ from $z_0$ to $z_1$ along a path $\gamma$:

1. Cover $\gamma$ with overlapping disks $D_0, D_1, \ldots, D_n$
2. On $D_0$: $f$ is given
3. On $D_0 \cap D_1$: use power series to define $f_1$ on $D_1$
4. Continue: each $f_{k+1}$ agrees with $f_k$ on $D_k \cap D_{k+1}$

```
Chain of disks along γ:

  z₀ ──────────────────── z₁
  •────•────•────•────•────•
  D₀   D₁   D₂   D₃   D₄   D₅

  Each Dₖ overlaps Dₖ₊₁
  Power series in each disk provides local continuation
```

### 3.2 Function Element

A **function element** is a pair $(f, D)$ where $f$ is analytic on a disk $D$. Analytic continuation is the process of chaining function elements along a path.

---

## 4. Monodromy Theorem

### 4.1 The Problem

Continuing along **different paths** from $z_0$ to $z_1$ can give **different values** (this is how multi-valued functions arise).

### 4.2 Statement

**Theorem (Monodromy).** If $f$ can be continued along every path in a simply connected domain $D$, and the continuation along two paths with the same endpoints gives the same result whenever the paths are homotopic in $D$, then $f$ extends to a **single-valued** analytic function on $D$.

More precisely: if $D$ is simply connected and $f$ admits analytic continuation along every path in $D$ starting at $z_0$, then the continuation defines a single-valued analytic function on $D$.

```
Simply connected domain ⟹ no monodromy ⟹ single-valued:

  ╭──────────╮                ╭──────────╮
 ╱   γ₁       ╲              ╱            ╲
│  z₀ ──→ z₁   │ Simply     │  z₀ ──→ z₁   │
│       ↑       │ connected: │  Same value   │
│  z₀ ──→ z₁   │ γ₁ ≃ γ₂   │  either path  │
 ╲   γ₂       ╱              ╲            ╱
  ╰──────────╯                ╰──────────╯

Multiply connected domain ⟹ possible monodromy:

  ╭──────────╮
 ╱  γ₁  ╭─╮  ╲
│  z₀ ──│•│→ z₁ │    Going around the hole may
│       ╰─╯     │    change the function value!
│  z₀ ────── z₁ │
 ╲   γ₂       ╱
  ╰──────────╯
```

---

## 5. Classical Examples

### 5.1 The Geometric Series → Meromorphic Function

$$f(z) = \sum_{n=0}^{\infty} z^n = \frac{1}{1-z}, \quad |z| < 1$$

The power series converges only in $|z| < 1$, but $1/(1-z)$ is defined on all of $\mathbb{C} \setminus \{1\}$. The rational function is the analytic continuation.

### 5.2 The Gamma Function

$$\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}\,dt, \quad \text{Re}(z) > 0$$

This formula defines $\Gamma$ only for $\text{Re}(z) > 0$. Using $\Gamma(z) = \Gamma(z+1)/z$, we continue to $\text{Re}(z) > -1$ (except $z = 0$), then to $\text{Re}(z) > -2$ (except $z = 0, -1$), and so on — to all of $\mathbb{C}$ minus $\{0, -1, -2, \ldots\}$.

### 5.3 The Riemann Zeta Function

$$\zeta(s) = \sum_{n=1}^{\infty} \frac{1}{n^s}, \quad \text{Re}(s) > 1$$

Via the functional equation:

$$\zeta(s) = 2^s \pi^{s-1}\sin\!\left(\frac{\pi s}{2}\right)\Gamma(1-s)\zeta(1-s)$$

$\zeta$ extends to a meromorphic function on all of $\mathbb{C}$ with a single simple pole at $s = 1$.

```
Riemann zeta function domains:

  Dirichlet series:        Analytic continuation:
  Re(s) > 1 only            All of ℂ (except s = 1)

     Im                          Im
      ▲                           ▲
      │    █████                  │█████████████
      │    █████                  │██████•██████
      ┼──┼─█████► Re             ┼──────┼──────► Re
      │    █████                  │██████ ██████
      │    █████                  │█████████████
         s=1                           s=1
                                  (simple pole)
```

---

## 6. Natural Boundaries

Not every function can be continued beyond its initial disk.

### 6.1 Lacunary Series

$$f(z) = \sum_{n=0}^{\infty} z^{2^n} = z + z^2 + z^4 + z^8 + z^{16} + \cdots$$

This converges on $|z| < 1$ and **cannot** be continued past any point of $|z| = 1$. The unit circle is a **natural boundary**.

### 6.2 Criterion

**Hadamard's Gap Theorem.** If $\sum a_n z^{n_k}$ with $n_{k+1}/n_k \geq \lambda > 1$ for all $k$, then the circle of convergence is a natural boundary.

---

## 7. Riemann Surfaces

### 7.1 Idea

When analytic continuation along different paths gives different values, the function is **multi-valued**. The remedy: build a **Riemann surface** — a branched covering of $\mathbb{C}$ on which the function becomes single-valued.

### 7.2 Example: $\sqrt{z}$

$\sqrt{z}$ has two branches. The Riemann surface is a two-sheeted cover of $\mathbb{C} \setminus \{0\}$:

```
Riemann surface for √z:

  Sheet 1:  +√z            ╭────────╮
                           ╱  Sheet 1 ╲
  Sheet 2:  -√z          │     ↕      │
                           ╲  Sheet 2 ╱
  Branch point at 0         ╰────────╯

  Going around z = 0 once: jump from Sheet 1 to Sheet 2
  Going around again: return to Sheet 1
```

### 7.3 Example: $\log z$

$\log z$ has infinitely many branches ($\log z + 2\pi i n$). The Riemann surface is an infinite helicoid (spiral staircase).

---

## 8. Design Example

### Example: Continue $f(z) = \sum_{n=1}^{\infty} z^n/n = -\log(1-z)$ beyond $|z| < 1$.

**Step 1.** The series converges for $|z| < 1$ and represents $-\log(1-z)$ (principal branch).

**Step 2.** $-\log(1-z)$ is analytic on $\mathbb{C} \setminus [1, \infty)$ (using the principal branch of $\log$).

**Step 3.** Continuing around $z = 1$: after one loop, $\log(1-z)$ changes by $2\pi i$.

```
Continuation of -log(1-z):

  Original series:           Analytic continuation:
  |z| < 1                     ℂ \ [1, ∞)

   ╭──╮                         ████████████
  ╱    ╲                        ████████████
 │  ○   │                      ████████████ ───── branch cut
  ╲    ╱                        ████████████
   ╰──╯                         ████████████
                                     1
```

**Result:** The analytic continuation of $\sum z^n/n$ is $-\log(1-z)$, defined on $\mathbb{C} \setminus [1, \infty)$ as a single-valued function (with principal branch), or on the Riemann surface of $\log$ as a multi-valued function.

---

## 9. Real-World Applications

| Application | How Analytic Continuation is Used |
|-------------|----------------------------------|
| **Number theory** | Extension of $\zeta(s)$, $L$-functions |
| **Quantum field theory** | Wick rotation, dimensional regularization |
| **Statistical mechanics** | Free energy as continued partition function |
| **Special functions** | $\Gamma$, Bessel, hypergeometric beyond initial domain |
| **Scattering theory** | $S$-matrix continuation to complex energies |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Analytic continuation | Extending $f$ beyond original domain preserving analyticity |
| Uniqueness | By Identity Theorem: continuation is unique |
| Chain of disks | Cover path with overlapping disks, match on overlaps |
| Monodromy | Simply connected $\Rightarrow$ continuation is single-valued |
| Multi-valuedness | Different paths may yield different values → Riemann surface |
| Natural boundary | Some functions cannot be continued past $\|z\| = R$ at all |
| Lacunary series | Hadamard gaps ⟹ natural boundary |
| Key examples | $1/(1-z)$, $\Gamma(s)$, $\zeta(s)$, $\log z$, $\sqrt{z}$ |

---

## Quick Revision Questions

1. **Define** analytic continuation and state the uniqueness theorem.

2. **Explain** the method of continuation along a chain of disks.

3. **State** the Monodromy Theorem and explain why simple connectivity is needed.

4. **Show** how $\Gamma(z)$ is analytically continued from $\text{Re}(z) > 0$ to $\mathbb{C} \setminus \{0, -1, -2, \ldots\}$.

5. **What** is a natural boundary? Give an example of a power series with the unit circle as natural boundary.

6. **Explain** how the Riemann surface of $\log z$ resolves the multi-valuedness of the logarithm.

---

[← Previous: Schwarz Lemma](04-schwarz-lemma.md) | [Back to README →](../README.md)
