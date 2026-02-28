# Chapter 5.1: Types of Functions

[← Previous: Lattices](../04-Relations/06-lattices.md) | [Back to README](../README.md) | [Next: Composition and Inverse →](02-composition-and-inverse.md)

---

## 📋 Chapter Overview

A **function** (or mapping) is a special type of relation that assigns exactly one output to each input. Functions are the backbone of mathematics and computer science — from arithmetic to algorithms.

---

## 1. Definition of a Function

A **function** $f: A \to B$ is a relation from $A$ to $B$ such that:

$$\forall a \in A, \; \exists! \, b \in B, \; (a, b) \in f$$

- $A$ = **domain** (set of inputs)
- $B$ = **codomain** (set of possible outputs)
- $f(a) = b$ means $a$ maps to $b$
- **Range** (image) = $\{f(a) : a \in A\} \subseteq B$

```
  Function:               NOT a function:
  
  A        B              A        B
  ┌──┐    ┌──┐            ┌──┐    ┌──┐
  │ 1│───→│ a│            │ 1│──→│ a│
  │  │    │  │            │  │╲   │  │
  │ 2│───→│ b│            │ 2│──→│ b│   1 maps to TWO
  │  │    │  │            │  │    │  │   values → NOT
  │ 3│───→│ c│            │ 3│    │ c│   a function
  └──┘    └──┘            └──┘    └──┘
                                         (3 has no image → 
                                          also not a function)
```

---

## 2. Injective (One-to-One) Functions

A function $f: A \to B$ is **injective** if different inputs give different outputs:

$$f(a_1) = f(a_2) \Rightarrow a_1 = a_2$$

Equivalently: $a_1 \neq a_2 \Rightarrow f(a_1) \neq f(a_2)$

```
  Injective:              NOT Injective:
  
  A        B              A        B
  ┌──┐    ┌──┐            ┌──┐    ┌──┐
  │ 1│───→│ a│            │ 1│──→│ a│
  │  │    │  │            │  │    │  │
  │ 2│───→│ b│            │ 2│──→│ a│  ← same output!
  │  │    │  │            │  │    │  │
  │ 3│───→│ c│            │ 3│──→│ b│
  │  │    │  │            └──┘    │  │
  └──┘    │ d│                    │ c│
          └──┘                    └──┘
  Each output used        'a' used twice
  at most once            → NOT injective
```

**Test:** Check that no horizontal line crosses the graph more than once (horizontal line test).

**Counting:** Number of injections from $A$ to $B$ (where $|A| = m, |B| = n$):

$$P(n, m) = \frac{n!}{(n-m)!} \quad \text{(requires } m \leq n \text{)}$$

---

## 3. Surjective (Onto) Functions

A function $f: A \to B$ is **surjective** if every element of $B$ is mapped to:

$$\forall b \in B, \; \exists a \in A, \; f(a) = b$$

Equivalently: Range = Codomain.

```
  Surjective:             NOT Surjective:
  
  A        B              A        B
  ┌──┐    ┌──┐            ┌──┐    ┌──┐
  │ 1│──→│ a│            │ 1│──→│ a│
  │  │    │  │            │  │    │  │
  │ 2│──→│ b│            │ 2│──→│ b│
  │  │╲   │  │            │  │    │  │
  │ 3│──→│ b│            │ 3│──→│ a│
  └──┘    └──┘            └──┘    │  │
                                   │ c│ ← never reached!
  Every element of B               └──┘
  is hit
```

**Requirement:** $|A| \geq |B|$ for surjection to be possible.

---

## 4. Bijective (One-to-One and Onto)

A function is **bijective** if it is both injective and surjective.

$$\text{Bijection} = \text{Injection} + \text{Surjection}$$

```
  Bijective:
  
  A        B
  ┌──┐    ┌──┐
  │ 1│───→│ a│     Perfect 1-to-1 pairing
  │  │    │  │     
  │ 2│───→│ b│     |A| = |B| required
  │  │    │  │
  │ 3│───→│ c│     Every input → unique output
  └──┘    └──┘     Every output ← unique input
```

**Key:** A bijection exists between $A$ and $B$ iff $|A| = |B|$.

**Counting:** Number of bijections from $A$ to $A$ (permutations) = $|A|!$

---

## 5. Classification Summary

| Type | Definition | Requirement |
|------|-----------|------------|
| **Injective** | $f(a_1) = f(a_2) \Rightarrow a_1 = a_2$ | $\|A\| \leq \|B\|$ |
| **Surjective** | $\forall b \in B, \exists a: f(a) = b$ | $\|A\| \geq \|B\|$ |
| **Bijective** | Injective + Surjective | $\|A\| = \|B\|$ |
| **General** | Just a valid function | $\|A\|, \|B\|$ any |

---

## 6. Important Function Families

| Function | Domain → Codomain | Type |
|----------|-------------------|------|
| $f(x) = x^2$ | $\mathbb{R} \to \mathbb{R}$ | Neither ($f(-1)=f(1)$, negative reals not in range) |
| $f(x) = x^2$ | $\mathbb{R}^+ \to \mathbb{R}^+$ | Bijective |
| $f(x) = 2x$ | $\mathbb{Z} \to \mathbb{Z}$ | Injective, not surjective |
| $f(x) = \lfloor x/2 \rfloor$ | $\mathbb{Z} \to \mathbb{Z}$ | Surjective, not injective |
| $f(x) = x + 1$ | $\mathbb{Z} \to \mathbb{Z}$ | Bijective |
| $f(x) = e^x$ | $\mathbb{R} \to \mathbb{R}^+$ | Bijective |

---

## 7. Special Functions

### Floor and Ceiling

$$\lfloor x \rfloor = \text{greatest integer} \leq x$$
$$\lceil x \rceil = \text{least integer} \geq x$$

| $x$ | $\lfloor x \rfloor$ | $\lceil x \rceil$ |
|:---:|:---:|:---:|
| 3.7 | 3 | 4 |
| -2.3 | -3 | -2 |
| 5 | 5 | 5 |

### Identity Function

$$\text{id}_A: A \to A, \quad \text{id}_A(a) = a$$

### Constant Function

$$f: A \to B, \quad f(a) = c \text{ for all } a \in A$$

---

## 8. Counting Functions

| From $A$ ($m$ elts) to $B$ ($n$ elts) | Count |
|---------------------------------------|-------|
| Total functions | $n^m$ |
| Injections ($m \leq n$) | $\frac{n!}{(n-m)!}$ |
| Bijections ($m = n$) | $n!$ |
| Surjections | $\sum_{k=0}^{n} (-1)^k \binom{n}{k}(n-k)^m$ |

**Example:** $|A| = 3, |B| = 2$:
- Total functions: $2^3 = 8$
- Surjections: $\binom{2}{0}(2)^3 - \binom{2}{1}(1)^3 + \binom{2}{2}(0)^3 = 8 - 2 + 0 = 6$

---

## 9. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │         Functions in Practice                     │
  │                                                   │
  │  1. Hash Functions (Surjective goal)             │
  │     Map keys → buckets; ideally uniform          │
  │                                                   │
  │  2. Encryption (Bijective)                       │
  │     Encrypt: plaintext → ciphertext (bijection)  │
  │     Decrypt: inverse function                     │
  │                                                   │
  │  3. Database Primary Keys (Injective)            │
  │     Each record → unique ID                       │
  │                                                   │
  │  4. Compression                                   │
  │     Lossless = injective; Lossy = not injective  │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Function $f: A \to B$ | Each $a \in A$ maps to exactly one $b \in B$ |
| Injective | No two inputs share an output |
| Surjective | Every output is used |
| Bijective | Both injective and surjective |
| Domain | Set of inputs |
| Codomain | Set of possible outputs |
| Range | Set of actual outputs ($\subseteq$ codomain) |
| $n^m$ | Number of functions from $m$-set to $n$-set |

---

## ❓ Quick Revision Questions

1. **Determine if $f(x) = 3x + 1$ from $\mathbb{R}$ to $\mathbb{R}$ is injective, surjective, or bijective.**

2. **How many functions are there from $\{1,2,3\}$ to $\{a,b\}$? How many are surjective?**

3. **Give an example of a function that is surjective but not injective.**

4. **If $f: A \to B$ is a bijection and $|A| = 5$, what is $|B|$?**

5. **Is the function $f(n) = n \mod 5$ from $\mathbb{Z}$ to $\{0,1,2,3,4\}$ injective? Surjective?**

6. **Explain the difference between codomain and range with an example.**

---

[← Previous: Lattices](../04-Relations/06-lattices.md) | [Back to README](../README.md) | [Next: Composition and Inverse →](02-composition-and-inverse.md)
