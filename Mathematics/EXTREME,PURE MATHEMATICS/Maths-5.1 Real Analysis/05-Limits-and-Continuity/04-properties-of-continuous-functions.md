# Chapter 4: Properties of Continuous Functions

[← Previous: Continuity — Definition](03-continuity-definition.md) | [Back to README](../README.md) | [Next: Intermediate Value Theorem →](05-intermediate-value-theorem.md)

---

## Overview

Continuous functions preserve algebraic operations and key topological properties. They map compact sets to compact sets, connected sets to connected sets, and satisfy important boundedness and sign-preservation results.

---

## 4.1 Algebraic Properties

**Theorem 4.1.** If $f, g$ are continuous at $c$, then so are:
- $f + g$, $f - g$, $kf$ (scalar multiple)
- $fg$ (product)
- $f/g$ (quotient, if $g(c) \neq 0$)
- $|f|$ (absolute value)
- $\max(f, g)$ and $\min(f, g)$

*Proof:* Follows from ALT for limits. $\blacksquare$

**Theorem 4.2.** If $f$ is continuous at $c$ and $g$ is continuous at $f(c)$, then $g \circ f$ is continuous at $c$.

*Proof (Sequential):* If $x_n \to c$, then $f(x_n) \to f(c)$ (continuity of $f$), so $g(f(x_n)) \to g(f(c))$ (continuity of $g$). $\blacksquare$

---

## 4.2 Continuous Functions and Compact Sets

**Theorem 4.3 (Preservation of Compactness).** If $K$ is compact and $f: K \to \mathbb{R}$ is continuous, then $f(K)$ is compact.

**Proof.** Let $\{O_\alpha\}$ be an open cover of $f(K)$. Then $\{f^{-1}(O_\alpha)\}$ is an open cover of $K$ (by topological continuity). By compactness of $K$, extract finite subcover $f^{-1}(O_{\alpha_1}), \ldots, f^{-1}(O_{\alpha_N})$. Then $O_{\alpha_1}, \ldots, O_{\alpha_N}$ cover $f(K)$. $\blacksquare$

```
  Compact → Continuous → Compact

  K = [a,b] ──f──► f(K) = [min, max]
  (compact)  cont.  (compact: closed + bounded!)

  This is why continuous functions on [a,b]:
  ✅ Are bounded
  ✅ Attain their bounds
```

---

## 4.3 Continuous Functions and Connected Sets

**Theorem 4.4 (Preservation of Connectedness).** If $E$ is connected and $f: E \to \mathbb{R}$ is continuous, then $f(E)$ is connected.

This + "connected in $\mathbb{R}$ = interval" gives the IVT (next chapter).

---

## 4.4 Sign Preservation and Boundedness

**Theorem 4.5 (Local Sign Preservation).** If $f$ is continuous at $c$ and $f(c) > 0$, then $\exists\, \delta > 0$ such that $f(x) > 0$ for all $x \in (c - \delta, c + \delta) \cap A$.

**Theorem 4.6 (Boundedness on Compact Sets).** If $f: K \to \mathbb{R}$ is continuous and $K$ is compact, then $f$ is bounded on $K$: $\exists\, M$ with $|f(x)| \leq M$ for all $x \in K$.

*Proof:* $f(K)$ is compact, hence bounded. $\blacksquare$

---

## 4.5 Monotone Functions and Continuity

**Theorem 4.7.** If $f: [a,b] \to \mathbb{R}$ is monotone, then:
1. $f$ can only have **jump discontinuities** (left and right limits exist at every point)
2. The set of discontinuities of $f$ is **at most countable**

**Proof sketch (2).** At each discontinuity $c$, the jump $f(c^+) - f(c^-)$ determines an open interval $(f(c^-), f(c^+))$ containing a rational. Different discontinuities give disjoint intervals (by monotonicity), hence disjoint rationals. So the set is countable. $\blacksquare$

```
  Monotone function: only jump discontinuities

  f(x) ▲
       │         ●────────
       │         │
       │  ●──────○ ← jump at c₂
       │  │
       │──○        ← jump at c₁
       │
       └──┼──────┼──────────► x
          c₁     c₂

  Each jump "uses up" a rational number → countably many jumps
```

---

## 4.6 Inverse Functions

**Theorem 4.8.** If $f: [a,b] \to \mathbb{R}$ is continuous and strictly monotone, then:
1. $f$ is a bijection onto $[f(a), f(b)]$ (or $[f(b), f(a)]$)
2. $f^{-1}$ exists and is continuous on $f([a,b])$

```
  Continuous + strictly monotone → invertible + inverse continuous

  f ──────────►  f⁻¹
  [a,b] ──────► [f(a),f(b)]
  continuous     continuous
  str. increasing  str. increasing
```

---

## Summary Table

| Property | Result |
|----------|--------|
| Algebraic operations | $f \pm g$, $fg$, $f/g$, $g \circ f$ continuous |
| Compact sets | $f(K)$ compact if $K$ compact, $f$ continuous |
| Connected sets | $f(E)$ connected if $E$ connected, $f$ continuous |
| Bounded on compact | Continuous $f$ on compact $K$ is bounded |
| Monotone discontinuities | At most countable, all jumps |
| Inverse function | Continuous, strictly monotone $\Rightarrow$ continuous inverse |

---

## Quick Revision Questions

1. **Prove** that the composition of continuous functions is continuous.

2. **Prove** that continuous images of compact sets are compact.

3. **Why** is $f(x) = 1/x$ unbounded on $(0, 1)$? Doesn't this contradict the boundedness theorem?

4. **Prove** that a monotone function on $[a,b]$ has at most countably many discontinuities.

5. **State and prove** the inverse function theorem for continuous monotone functions.

6. **True or False:** The image of an open set under a continuous function is open. Justify.

---

[← Previous: Continuity — Definition](03-continuity-definition.md) | [Back to README](../README.md) | [Next: Intermediate Value Theorem →](05-intermediate-value-theorem.md)
