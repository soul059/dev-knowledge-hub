# Chapter 5.2: Composition and Inverse Functions

[← Previous: Types of Functions](01-types-of-functions.md) | [Back to README](../README.md) | [Next: Pigeonhole Principle →](03-pigeonhole-principle.md)

---

## 📋 Chapter Overview

**Composition** chains functions together, and **inverse functions** undo what a function does. These concepts are essential for understanding function algebra, cryptography, and algorithm design.

---

## 1. Function Composition

If $f: A \to B$ and $g: B \to C$, the **composition** $g \circ f: A \to C$ is:

$$(g \circ f)(x) = g(f(x))$$

```
  Composition Pipeline:
  
     x ──→ [  f  ] ──→ f(x) ──→ [  g  ] ──→ g(f(x))
     
     A ─────────→ B ──────────→ C
              f          g
              
         g ∘ f: A ──────────────→ C
```

**Important:** $g \circ f$ means "apply $f$ first, then $g$" (right to left!).

---

## 2. Worked Examples

### Example 1
$f: \mathbb{R} \to \mathbb{R}, \; f(x) = 2x + 1$
$g: \mathbb{R} \to \mathbb{R}, \; g(x) = x^2$

$$(g \circ f)(x) = g(f(x)) = g(2x + 1) = (2x + 1)^2$$
$$(f \circ g)(x) = f(g(x)) = f(x^2) = 2x^2 + 1$$

**Note:** $g \circ f \neq f \circ g$ in general! Composition is **not commutative**.

### Example 2 (Finite Sets)

$A = \{1, 2, 3\}$, $f = \{(1,2),(2,3),(3,1)\}$, $g = \{(1,3),(2,1),(3,2)\}$

| $x$ | $f(x)$ | $g(f(x))$ |
|:---:|:------:|:---------:|
| 1 | 2 | $g(2) = 1$ |
| 2 | 3 | $g(3) = 2$ |
| 3 | 1 | $g(1) = 3$ |

$$g \circ f = \{(1,1),(2,2),(3,3)\} = \text{id}$$

---

## 3. Properties of Composition

| Property | Status |
|----------|:------:|
| Associative | ✅ $h \circ (g \circ f) = (h \circ g) \circ f$ |
| Commutative | ❌ Generally $g \circ f \neq f \circ g$ |
| Identity | ✅ $f \circ \text{id}_A = f = \text{id}_B \circ f$ |

### Composition Preserves Types

| If $f$ is | and $g$ is | then $g \circ f$ is |
|-----------|-----------|---------------------|
| Injective | Injective | Injective |
| Surjective | Surjective | Surjective |
| Bijective | Bijective | Bijective |

---

## 4. Inverse Functions

If $f: A \to B$ is a **bijection**, then $f^{-1}: B \to A$ exists and satisfies:

$$f^{-1}(f(a)) = a \quad \text{for all } a \in A$$
$$f(f^{-1}(b)) = b \quad \text{for all } b \in B$$

Equivalently:

$$f^{-1} \circ f = \text{id}_A \quad \text{and} \quad f \circ f^{-1} = \text{id}_B$$

```
  f: A → B (bijection)        f⁻¹: B → A
  
  A        B                   B        A
  ┌──┐    ┌──┐                ┌──┐    ┌──┐
  │ 1│──→│ a│                │ a│──→│ 1│
  │  │    │  │                │  │    │  │
  │ 2│──→│ b│                │ b│──→│ 2│
  │  │    │  │                │  │    │  │
  │ 3│──→│ c│                │ c│──→│ 3│
  └──┘    └──┘                └──┘    └──┘
  
  f⁻¹ ∘ f = id_A              f ∘ f⁻¹ = id_B
```

---

## 5. Finding Inverses

### Algebraic Method

To find $f^{-1}$ for $f(x) = 2x + 3$:

1. Write $y = 2x + 3$
2. Solve for $x$: $x = \frac{y - 3}{2}$
3. Therefore $f^{-1}(y) = \frac{y - 3}{2}$

**Verification:** $f^{-1}(f(x)) = \frac{(2x+3) - 3}{2} = \frac{2x}{2} = x$ ✓

### Table/Set Method

If $f = \{(1,a),(2,b),(3,c)\}$, then $f^{-1} = \{(a,1),(b,2),(c,3)\}$.

Just swap all ordered pairs.

---

## 6. When Does the Inverse Exist?

| Type of $f$ | $f^{-1}$ exists? |
|-------------|:-:|
| Bijective | ✅ Yes (unique) |
| Injective only | ❌ No (not surjective ⇒ some $b$ have no preimage) |
| Surjective only | ❌ No (not injective ⇒ some $b$ have multiple preimages) |
| Neither | ❌ No |

### Left and Right Inverses

Even non-bijective functions can have **one-sided** inverses:

- **Left inverse** of $f$: a function $g$ with $g \circ f = \text{id}_A$ (exists iff $f$ is injective)
- **Right inverse** of $f$: a function $h$ with $f \circ h = \text{id}_B$ (exists iff $f$ is surjective)

---

## 7. Properties of Inverses

| Property | Formula |
|----------|---------|
| $(f^{-1})^{-1} = f$ | Inverse of inverse |
| $(g \circ f)^{-1} = f^{-1} \circ g^{-1}$ | "Socks and shoes" rule |
| $\text{id}^{-1} = \text{id}$ | Identity is its own inverse |

**The "Socks-and-Shoes" Rule explained:**

```
  Putting on:   Socks first, then Shoes
                f = socks,  g = shoes
                g ∘ f = fully dressed
  
  Taking off:   Shoes first, then Socks
                f⁻¹ ∘ g⁻¹ = undressing
                
  (g ∘ f)⁻¹ = f⁻¹ ∘ g⁻¹    (reverse order!)
```

---

## 8. Permutations as Bijections

A **permutation** of $\{1, 2, \ldots, n\}$ is a bijection from the set to itself.

**Two-line notation:**

$$\sigma = \begin{pmatrix} 1 & 2 & 3 & 4 \\ 3 & 1 & 4 & 2 \end{pmatrix}$$

means $\sigma(1)=3, \sigma(2)=1, \sigma(3)=4, \sigma(4)=2$.

**Cycle notation:** $\sigma = (1 \; 3 \; 4 \; 2)$

**Inverse:** $\sigma^{-1} = \begin{pmatrix} 1 & 2 & 3 & 4 \\ 2 & 4 & 1 & 3 \end{pmatrix} = (1 \; 2 \; 4 \; 3)$

```
  σ:     1 → 3 → 4 → 2 → 1  (cycle)
  σ⁻¹:   1 → 2 → 4 → 3 → 1  (reverse cycle)
```

---

## 9. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │         Composition & Inverses in Practice        │
  │                                                   │
  │  1. Encryption/Decryption                        │
  │     Encrypt = f, Decrypt = f⁻¹                   │
  │     f must be bijective for decryption to work!  │
  │                                                   │
  │  2. Pipeline Processing                          │
  │     Data flows through stages: g ∘ f ∘ h         │
  │     Unix pipes: cmd1 | cmd2 | cmd3               │
  │                                                   │
  │  3. Undo/Redo in Editors                         │
  │     Action = f, Undo = f⁻¹                       │
  │     Stack of compositions                        │
  │                                                   │
  │  4. Coordinate Transformations                   │
  │     Rotate, translate, scale = composed functions │
  │     Inverse = undo transformation                │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| $(g \circ f)(x) = g(f(x))$ | Apply $f$ first, then $g$ |
| Associative | $(h \circ g) \circ f = h \circ (g \circ f)$ |
| Not commutative | $g \circ f \neq f \circ g$ generally |
| $f^{-1}$ exists | iff $f$ is bijective |
| $f^{-1} \circ f = \text{id}$ | Inverse undoes the function |
| $(g \circ f)^{-1} = f^{-1} \circ g^{-1}$ | Socks-and-shoes rule |
| Permutation | Bijection from a set to itself |

---

## ❓ Quick Revision Questions

1. **If $f(x) = x + 3$ and $g(x) = 2x$, compute $g \circ f$ and $f \circ g$.**

2. **Find the inverse of $f(x) = \frac{x - 5}{3}$.**

3. **Prove that if $f$ and $g$ are injective, then $g \circ f$ is injective.**

4. **Why does the function $f(x) = x^2$ from $\mathbb{R} \to \mathbb{R}$ not have an inverse?**

5. **Compute the composition and inverse of $\sigma = (1\;2\;3)$ and $\tau = (2\;3)$.**

6. **Explain the "socks and shoes" property with a concrete example.**

---

[← Previous: Types of Functions](01-types-of-functions.md) | [Back to README](../README.md) | [Next: Pigeonhole Principle →](03-pigeonhole-principle.md)
