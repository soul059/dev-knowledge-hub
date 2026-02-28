# Chapter 8.2 — Algebraic and Transcendental Extensions

[← Previous: Field Extensions](01-field-extensions.md) | [Back to README](../README.md) | [Next: Finite Fields →](03-finite-fields.md)

---

## Overview

Every element of an extension field is either **algebraic** (root of a polynomial over the base field) or **transcendental** (not a root of any such polynomial). This dichotomy drives the structure of field extensions and connects algebra to number theory and analysis.

---

## 1. Algebraic Elements

**Definition.** Let $E/F$ be a field extension. An element $\alpha \in E$ is **algebraic over $F$** if there exists a non-zero polynomial $f(x) \in F[x]$ such that $f(\alpha) = 0$.

An element is **transcendental over $F$** if it is not algebraic.

**Examples:**

| Element | Over $\mathbb{Q}$ | Reason |
|---------|-------------------|--------|
| $\sqrt{2}$ | Algebraic | Root of $x^2 - 2$ |
| $\sqrt[3]{5}$ | Algebraic | Root of $x^3 - 5$ |
| $i$ | Algebraic | Root of $x^2 + 1$ |
| $\pi$ | Transcendental | Lindemann (1882) |
| $e$ | Transcendental | Hermite (1873) |
| $\sqrt{2} + \sqrt{3}$ | Algebraic | Root of $x^4 - 10x^2 + 1$ |
| $e^\pi$ | Transcendental | Gelfond–Schneider |

---

## 2. The Minimal Polynomial

**Theorem 8.2.** If $\alpha$ is algebraic over $F$, then there exists a unique monic irreducible polynomial $m_\alpha(x) \in F[x]$ such that $m_\alpha(\alpha) = 0$. This is the **minimal polynomial** of $\alpha$ over $F$.

Properties:
1. $m_\alpha$ divides any $f \in F[x]$ with $f(\alpha) = 0$.
2. $m_\alpha = $ generator of $\ker(\text{ev}_\alpha: F[x] \to E)$.
3. $\deg(m_\alpha) = [F(\alpha) : F]$.

*Proof.* The evaluation map $\text{ev}_\alpha: F[x] \to E$, $f \mapsto f(\alpha)$, has kernel $\ker(\text{ev}_\alpha) = (m_\alpha)$ since $F[x]$ is a PID. Since $\text{im}(\text{ev}_\alpha) = F[\alpha] \cong F[x]/(m_\alpha)$ must be a domain sitting inside a field, $(m_\alpha)$ is prime, so $m_\alpha$ is irreducible. Monic by normalisation. Uniqueness is clear. $\blacksquare$

---

## 3. Computing Minimal Polynomials

**Strategy:**
1. Guess $m_\alpha$ by finding relations among powers of $\alpha$.
2. Verify irreducibility.
3. Check $m_\alpha(\alpha) = 0$.

**Example 1:** $\alpha = \sqrt{2} + \sqrt{3}$ over $\mathbb{Q}$.

$$\alpha^2 = 5 + 2\sqrt{6} \implies \alpha^2 - 5 = 2\sqrt{6} \implies (\alpha^2 - 5)^2 = 24 \implies \alpha^4 - 10\alpha^2 + 1 = 0$$

So $m_\alpha(x) = x^4 - 10x^2 + 1$. Check irreducibility: Eisenstein doesn't apply directly. Mod 2: $x^4 + 1$, which factors as $(x+1)^4$ over $\mathbb{F}_2$. Mod 3: $x^4 - x^2 + 1$; check roots: $0,1,2$ give $1, 1, 1$ — no roots. Check quadratic factors over $\mathbb{F}_3$: $(x^2 + ax + b)(x^2 - ax + b^{-1} \cdot 1)$... After checking, it's irreducible over $\mathbb{Q}$.

**Example 2:** $\alpha = \sqrt[3]{2}$ over $\mathbb{Q}$.

$m_\alpha(x) = x^3 - 2$ (irreducible by Eisenstein with $p = 2$).

```
  Finding min poly of α = √2 + √3:
  
  α = √2 + √3
  α² = 5 + 2√6
  α² - 5 = 2√6
  (α² - 5)² = 24
  α⁴ - 10α² + 25 = 24
  α⁴ - 10α² + 1 = 0
  
  m_α(x) = x⁴ - 10x² + 1,  degree 4
  ∴ [ℚ(√2+√3) : ℚ] = 4
```

---

## 4. Algebraic Extensions

**Definition.** $E/F$ is an **algebraic extension** if every $\alpha \in E$ is algebraic over $F$.

**Theorem 8.3.** Every finite extension is algebraic.

*Proof.* If $[E:F] = n$, then for any $\alpha \in E$, the $n+1$ elements $1, \alpha, \alpha^2, \ldots, \alpha^n$ are $F$-linearly dependent, giving a polynomial relation. $\blacksquare$

**Converse is false:** The algebraic closure $\overline{\mathbb{Q}}/\mathbb{Q}$ is algebraic but infinite.

---

## 5. Transcendental Extensions

**Theorem 8.4.** $\alpha$ is transcendental over $F$ $\iff$ $F[\alpha] \cong F[x]$ (polynomial ring) $\iff$ $F(\alpha) \cong F(x)$ (rational function field).

*Proof.* $\alpha$ transcendental $\iff$ $\ker(\text{ev}_\alpha) = \{0\}$ $\iff$ $\text{ev}_\alpha$ is injective $\iff$ $F[\alpha] \cong F[x]$. Taking fraction fields: $F(\alpha) \cong F(x)$. $\blacksquare$

**Important:** $[F(x) : F] = \infty$, so transcendental extensions are always infinite.

Transcendence theory is deep:
- **Lindemann–Weierstrass:** $e^{\alpha_1}, \ldots, e^{\alpha_n}$ algebraically independent if $\alpha_1, \ldots, \alpha_n$ algebraically independent algebraic numbers.
- **Schanuel's conjecture:** vast open generalisation.

---

## 6. Algebraic Closure

**Definition.** A field $F$ is **algebraically closed** if every non-constant $f \in F[x]$ has a root in $F$.

Equivalently: every $f \in F[x]$ splits completely into linear factors.

**Theorem 8.5 (Fundamental Theorem of Algebra).** $\mathbb{C}$ is algebraically closed.

**Definition.** An **algebraic closure** $\overline{F}$ of $F$ is:
1. An algebraic extension $\overline{F}/F$, and
2. $\overline{F}$ is algebraically closed.

**Theorem 8.6 (Steinitz).** Every field has an algebraic closure, unique up to isomorphism.

```
  Containment diagram:
  
  F̄  (algebraic closure, algebraically closed)
  │
  │  algebraic
  │
  E  (some algebraic extension)
  │
  │  
  F  (base field)
  
  Examples:
  ℚ̄ ⊂ ℂ    (algebraic numbers inside ℂ)
  F̄ₚ = ⋃ 𝔽_{pⁿ}   (union of all finite fields of char p)
```

---

## 7. Degree of Algebraic Numbers

**Theorem 8.7.** If $\alpha$ is algebraic over $F$ and $\beta$ is algebraic over $F(\alpha)$, then $\beta$ is algebraic over $F$.

*Proof.* $[F(\alpha,\beta):F] = [F(\alpha,\beta):F(\alpha)] \cdot [F(\alpha):F] < \infty$, so $F(\alpha,\beta)/F$ is finite, hence algebraic. $\blacksquare$

**Corollary.** The set of all algebraic numbers over $F$ within an extension $E$ forms a subfield.

$\overline{\mathbb{Q}} = \{\alpha \in \mathbb{C} : \alpha \text{ algebraic over } \mathbb{Q}\}$ is a field — the field of **algebraic numbers**.

---

## 8. Transcendence Degree

For purely transcendental extensions $F(x_1, \ldots, x_n)/F$:

**Transcendence degree** = number of algebraically independent transcendental elements needed.

$$\text{tr.deg}(\mathbb{R}/\mathbb{Q}) = |\mathbb{R}| = \mathfrak{c} \quad (\text{uncountable})$$

$$\text{tr.deg}(F(x,y)/F) = 2$$

A general extension decomposes: first a purely transcendental extension, then an algebraic extension on top (Lüroth, Noether).

---

## Summary Table

| Concept | Definition / Result |
|---------|-------------------|
| Algebraic element | Root of some $f \in F[x]$ |
| Transcendental element | Not algebraic |
| Minimal polynomial $m_\alpha$ | Unique monic irreducible with $m_\alpha(\alpha) = 0$ |
| $\deg m_\alpha$ | $= [F(\alpha):F]$ |
| Finite $\implies$ algebraic | Every finite extension is algebraic |
| Algebraic closure $\overline{F}$ | Algebraic + algebraically closed; exists, unique up to $\cong$ |
| Algebraic numbers | Form a subfield of $\mathbb{C}$ |
| Transcendence degree | Number of algebraically independent transcendentals |

---

## Quick Revision Questions

1. **Find** the minimal polynomial of $\sqrt{3} + \sqrt{5}$ over $\mathbb{Q}$ and determine $[\mathbb{Q}(\sqrt{3}+\sqrt{5}):\mathbb{Q}]$.

2. **Prove** that every finite extension is algebraic.

3. **Show** that $\pi + e$ or $\pi \cdot e$ (or both) must be transcendental over $\mathbb{Q}$.

4. **Prove** that the set of algebraic numbers forms a field.

5. **Find** the minimal polynomial of $i + \sqrt{2}$ over $\mathbb{Q}$.

6. **Explain** why $[F(\alpha):F] = \infty$ whenever $\alpha$ is transcendental over $F$.

---

[← Previous: Field Extensions](01-field-extensions.md) | [Back to README](../README.md) | [Next: Finite Fields →](03-finite-fields.md)
