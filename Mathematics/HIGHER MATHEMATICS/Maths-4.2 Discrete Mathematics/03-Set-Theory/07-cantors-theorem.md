# Chapter 3.7: Cantor's Theorem

[← Previous: Countable and Uncountable Sets](06-countable-and-uncountable-sets.md) | [Back to README](../README.md) | [Next: Definition and Representation of Relations →](../04-Relations/01-definition-and-representation.md)

---

## 📋 Chapter Overview

**Cantor's Theorem** is one of the most remarkable results in all of mathematics: for any set $A$, the power set $\mathcal{P}(A)$ is strictly larger than $A$. This means there is no "largest infinity" — there is always a bigger one.

---

## 1. Statement of the Theorem

**Cantor's Theorem:** For any set $A$,

$$|A| < |\mathcal{P}(A)|$$

In other words, there is **no surjection** from $A$ to $\mathcal{P}(A)$.

---

## 2. Proof

We prove two parts:

### Part 1: $|A| \leq |\mathcal{P}(A)|$

The function $f: A \to \mathcal{P}(A)$ defined by $f(a) = \{a\}$ is injective.

Therefore $|A| \leq |\mathcal{P}(A)|$.

### Part 2: $|A| \neq |\mathcal{P}(A)|$ (No bijection exists)

**Proof by contradiction.** Suppose $g: A \to \mathcal{P}(A)$ is a surjection. Define:

$$D = \{a \in A : a \notin g(a)\}$$

Since $D \subseteq A$, we have $D \in \mathcal{P}(A)$.

Since $g$ is surjective, there exists some $d \in A$ with $g(d) = D$.

**Question:** Is $d \in D$?

- If $d \in D$, then by definition of $D$, $d \notin g(d) = D$. **Contradiction!**
- If $d \notin D$, then $d \notin D$ means $d \notin \{a \in A : a \notin g(a)\}$, so $d \in g(d) = D$. **Contradiction!**

Both cases lead to contradiction, so no surjection exists. $\blacksquare$

```
  Cantor's Diagonal Construction
  ═══════════════════════════════
  
  Suppose g: A → P(A) is surjective.
  
  Element │ g(element) = subset of A     │ In own image?
  ────────┼──────────────────────────────┼──────────────
    a₁    │ {a₁, a₃, a₅}               │ a₁ ∈ g(a₁) ✓
    a₂    │ {a₄, a₅}                   │ a₂ ∉ g(a₂) ✗
    a₃    │ {a₁, a₂, a₃}              │ a₃ ∈ g(a₃) ✓
    a₄    │ {}                          │ a₄ ∉ g(a₄) ✗
    a₅    │ {a₂, a₅}                   │ a₅ ∈ g(a₅) ✓
    ⋮     │  ⋮                          │     ⋮
  
  D = {elements NOT in their own image}
    = {a₂, a₄, ...}
  
  D ∈ P(A), but D ≠ g(aᵢ) for any aᵢ!
  ⇒ g is NOT surjective. Contradiction!
```

---

## 3. Consequence: An Infinite Tower of Infinities

Applying Cantor's Theorem repeatedly:

$$|\mathbb{N}| < |\mathcal{P}(\mathbb{N})| < |\mathcal{P}(\mathcal{P}(\mathbb{N}))| < |\mathcal{P}(\mathcal{P}(\mathcal{P}(\mathbb{N})))| < \cdots$$

```
  Tower of Infinities
  ═══════════════════
  
  Level 0:  |ℕ|                    = ℵ₀
  Level 1:  |P(ℕ)|                 = 2^ℵ₀  = 𝔠
  Level 2:  |P(P(ℕ))|             = 2^𝔠
  Level 3:  |P(P(P(ℕ)))|         = 2^(2^𝔠)
     ⋮           ⋮                     ⋮
  
  Each level is STRICTLY larger than the previous!
  
  ℵ₀ < 2^ℵ₀ < 2^(2^ℵ₀) < 2^(2^(2^ℵ₀)) < ...
```

There is **no largest cardinal number** — for any infinity, there is always a bigger one.

---

## 4. Connection to the Continuum Hypothesis

**Continuum Hypothesis (CH):** There is no set whose cardinality is strictly between $|\mathbb{N}|$ and $|\mathbb{R}|$.

$$\aleph_0 < ??? < 2^{\aleph_0} \quad \text{(CH says no such set exists)}$$

**Result (Gödel, 1940 & Cohen, 1963):** The Continuum Hypothesis is **independent** of the standard axioms of set theory (ZFC). It can be neither proved nor disproved!

| Statement | Status |
|-----------|--------|
| $\|\mathbb{N}\| < \|\mathbb{R}\|$ | **Proved** (Cantor) |
| $\|\mathbb{R}\| = 2^{\aleph_0}$ | **Proved** |
| No set between $\aleph_0$ and $2^{\aleph_0}$ | **Independent of ZFC** |

---

## 5. Russell's Paradox Connection

Cantor's diagonal argument inspired **Russell's Paradox**:

"Let $R = \{x : x \notin x\}$. Is $R \in R$?"

```
  ┌──────────────────────────────────────────────┐
  │   Cantor's D = {a ∈ A : a ∉ g(a)}           │
  │                    ↕                          │
  │   Russell's R = {x : x ∉ x}                  │
  │                                               │
  │   Same self-referential structure!            │
  │   Both produce contradictions from the        │
  │   assumption that "everything is included."   │
  └──────────────────────────────────────────────┘
```

This paradox led to the development of **axiomatic set theory** (ZFC) to avoid such contradictions.

---

## 6. Generalization: Cantor's Theorem for Functions

**Corollary:** $|A| < |B^A|$ where $B^A$ denotes the set of all functions from $A$ to $B$ (when $|B| \geq 2$).

Since $\mathcal{P}(A) \cong \{0, 1\}^A$ (characteristic functions), this is a generalization.

| Sets | Result |
|------|--------|
| $\|A\| < \|\mathcal{P}(A)\|$ | Original theorem |
| $\|A\| < \|\{0,1\}^A\|$ | Equivalent form |
| $\|A\| < \|B^A\|$ for $\|B\| \geq 2$ | Generalization |

---

## 7. Worked Example

**Example:** Show that the set of all functions $f: \mathbb{N} \to \{0, 1\}$ is uncountable.

**Solution:**

By Cantor's Theorem: $|\mathbb{N}| < |\{0,1\}^{\mathbb{N}}|$

Since $|\mathbb{N}| = \aleph_0$, we have $|\{0,1\}^{\mathbb{N}}| > \aleph_0$, so $\{0,1\}^{\mathbb{N}}$ is uncountable.

**Direct connection:** Each function $f: \mathbb{N} \to \{0,1\}$ corresponds to a subset of $\mathbb{N}$:

$$f \leftrightarrow \{n \in \mathbb{N} : f(n) = 1\}$$

So $|\{0,1\}^{\mathbb{N}}| = |\mathcal{P}(\mathbb{N})| = 2^{\aleph_0} = \mathfrak{c}$.

```
  Function f          ↔    Subset S = {n : f(n) = 1}
  ─────────────────────────────────────────────────
  f = 1,0,1,1,0,...   ↔    S = {0, 2, 3, ...}
  f = 0,0,0,0,0,...   ↔    S = ∅
  f = 1,1,1,1,1,...   ↔    S = ℕ
  f = 1,0,1,0,1,...   ↔    S = {0, 2, 4, ...} (evens)
```

---

## 8. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │         Why Cantor's Theorem Matters              │
  │                                                   │
  │  1. Computability Theory                          │
  │     • Only countably many programs exist           │
  │     • Uncountably many decision problems           │
  │     → Most problems are unsolvable!               │
  │                                                   │
  │  2. Database Theory                               │
  │     • Query languages must be limited to           │
  │       computable functions                        │
  │                                                   │
  │  3. Foundations of Mathematics                     │
  │     • Motivates axiomatic set theory              │
  │     • Establishes hierarchy of mathematical        │
  │       objects                                     │
  │                                                   │
  │  4. Information Theory                            │
  │     • Connections to entropy and encoding limits   │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Cantor's Theorem | $\|A\| < \|\mathcal{P}(A)\|$ for any set $A$ |
| Proof technique | Diagonal argument (construct set $D$ not in range) |
| Tower of infinities | $\aleph_0 < 2^{\aleph_0} < 2^{2^{\aleph_0}} < \cdots$ |
| Continuum Hypothesis | No cardinality between $\aleph_0$ and $2^{\aleph_0}$ (independent of ZFC) |
| Russell's Paradox | Same self-referential structure as proof |
| Key consequence | No "set of all sets" can exist |

---

## ❓ Quick Revision Questions

1. **State Cantor's Theorem precisely.**

2. **In the proof, why does the set $D = \{a \in A : a \notin g(a)\}$ cause a contradiction?**

3. **How does Cantor's Theorem show there is no largest infinity?**

4. **What is the Continuum Hypothesis, and what is its status?**

5. **Explain the connection between Cantor's diagonal argument and Russell's Paradox.**

6. **Why does Cantor's Theorem imply that some mathematical problems are undecidable by computers?**

---

[← Previous: Countable and Uncountable Sets](06-countable-and-uncountable-sets.md) | [Back to README](../README.md) | [Next: Definition and Representation of Relations →](../04-Relations/01-definition-and-representation.md)
