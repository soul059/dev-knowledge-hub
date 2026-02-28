# Chapter 1.1 — Definition and Examples of Groups

[← Back to README](../README.md) | [Next: Order of an Element →](02-order-of-element.md)

---

## Overview

A **group** is the most fundamental algebraic structure studied in abstract algebra. It captures the idea of *symmetry* and *reversible operations*. In this chapter we define a group axiomatically, explore many examples, and learn to read **Cayley tables**.

---

## 1. Binary Operations

**Definition.** A **binary operation** on a set $S$ is a function $\ast : S \times S \to S$. We write $a \ast b$ instead of $\ast(a, b)$.

**Key property — Closure:** For all $a, b \in S$, the result $a \ast b$ is also in $S$.

### Examples of Binary Operations

| Set | Operation | Closed? |
|-----|-----------|---------|
| $\mathbb{Z}$ | Addition $+$ | Yes |
| $\mathbb{Z}$ | Subtraction $-$ | Yes |
| $\mathbb{N}$ | Subtraction $-$ | **No** ($2 - 5 = -3 \notin \mathbb{N}$) |
| $\mathbb{R} \setminus \{0\}$ | Multiplication $\cdot$ | Yes |
| $M_n(\mathbb{R})$ | Matrix multiplication | Yes |

---

## 2. The Group Axioms

**Definition.** A **group** is a pair $(G, \ast)$ where $G$ is a non-empty set and $\ast$ is a binary operation on $G$ satisfying:

| Axiom | Statement | Formal |
|-------|-----------|--------|
| **G1 — Closure** | $a \ast b \in G$ for all $a, b \in G$ | $\ast : G \times G \to G$ |
| **G2 — Associativity** | $(a \ast b) \ast c = a \ast (b \ast c)$ | $\forall\, a,b,c \in G$ |
| **G3 — Identity** | There exists $e \in G$ with $e \ast a = a \ast e = a$ | $\exists\, e \in G,\; \forall\, a \in G$ |
| **G4 — Inverses** | For each $a$, there exists $a^{-1}$ with $a \ast a^{-1} = a^{-1} \ast a = e$ | $\forall\, a \in G,\; \exists\, a^{-1} \in G$ |

If additionally $a \ast b = b \ast a$ for all $a, b \in G$, we call $(G, \ast)$ an **abelian group** (named after Niels Henrik Abel).

### Axiom Verification Flowchart

```
  Start with (S, ∗)
        │
        ▼
  ┌─────────────┐     No
  │  Closed?    ├──────────► Not a group
  └──────┬──────┘
         │ Yes
         ▼
  ┌──────────────┐    No
  │ Associative? ├─────────► Not a group
  └──────┬───────┘
         │ Yes
         ▼
  ┌──────────────┐    No
  │  Identity?   ├─────────► Not a group
  └──────┬───────┘
         │ Yes
         ▼
  ┌──────────────┐    No
  │  Inverses?   ├─────────► Not a group
  └──────┬───────┘
         │ Yes
         ▼
     ✓ GROUP
         │
         ▼
  ┌──────────────┐    No
  │ Commutative? ├─────────► Non-abelian group
  └──────┬───────┘
         │ Yes
         ▼
    Abelian Group
```

---

## 3. Fundamental Properties

**Theorem 1.1 (Uniqueness of identity).** A group has exactly one identity element.

*Proof.* Suppose $e$ and $e'$ are both identities. Then $e = e \ast e' = e'$. $\square$

**Theorem 1.2 (Uniqueness of inverses).** Each element has exactly one inverse.

*Proof.* Suppose $b$ and $c$ are both inverses of $a$. Then:
$$b = b \ast e = b \ast (a \ast c) = (b \ast a) \ast c = e \ast c = c. \quad \square$$

**Theorem 1.3 (Cancellation laws).** In a group:
- **Left cancellation:** $a \ast b = a \ast c \implies b = c$
- **Right cancellation:** $b \ast a = c \ast a \implies b = c$

**Theorem 1.4 (Socks-shoes property).** $(a \ast b)^{-1} = b^{-1} \ast a^{-1}$.

*Proof.*
$$(a \ast b) \ast (b^{-1} \ast a^{-1}) = a \ast (b \ast b^{-1}) \ast a^{-1} = a \ast e \ast a^{-1} = a \ast a^{-1} = e. \quad \square$$

---

## 4. Classic Examples of Groups

### 4.1 The Integers under Addition: $(\mathbb{Z}, +)$

| Axiom | Verification |
|-------|-------------|
| Closure | $a + b \in \mathbb{Z}$ ✓ |
| Associativity | $(a+b)+c = a+(b+c)$ ✓ |
| Identity | $e = 0$ since $0 + a = a$ ✓ |
| Inverses | $a^{-1} = -a$ since $a + (-a) = 0$ ✓ |
| Abelian? | Yes: $a + b = b + a$ ✓ |

### 4.2 Integers Modulo $n$: $(\mathbb{Z}_n, +)$

$\mathbb{Z}_n = \{0, 1, 2, \ldots, n-1\}$ with addition modulo $n$.

**Cayley table for $(\mathbb{Z}_4, +)$:**

```
  +  │  0   1   2   3
 ────┼────────────────
  0  │  0   1   2   3
  1  │  1   2   3   0
  2  │  2   3   0   1
  3  │  3   0   1   2
```

- Identity: $0$
- Inverse of $1$ is $3$ (since $1 + 3 = 0 \mod 4$)
- Inverse of $2$ is $2$ (since $2 + 2 = 0 \mod 4$)

### 4.3 Units Modulo $n$: $(\mathbb{Z}_n^\times, \cdot)$

$\mathbb{Z}_n^\times = \{a \in \mathbb{Z}_n : \gcd(a, n) = 1\}$ under multiplication mod $n$.

**Example: $\mathbb{Z}_8^\times = \{1, 3, 5, 7\}$**

```
  ×  │  1   3   5   7
 ────┼────────────────
  1  │  1   3   5   7
  3  │  3   1   7   5
  5  │  5   7   1   3
  7  │  7   5   3   1
```

- Identity: $1$
- Every element is its own inverse! (Each diagonal entry is $1$.)
- This group is abelian.
- $|\mathbb{Z}_n^\times| = \varphi(n)$ (Euler's totient function).

### 4.4 The Symmetric Group $S_n$

$S_n$ is the set of all bijections $\sigma: \{1, 2, \ldots, n\} \to \{1, 2, \ldots, n\}$, under composition.

- $|S_n| = n!$
- Non-abelian for $n \geq 3$.

**Elements of $S_3$ in cycle notation:**

| Permutation | Cycle Notation | Type |
|------------|----------------|------|
| $e$ | $(1)(2)(3)$ | Identity |
| $\sigma_1$ | $(1\;2)$ | Transposition |
| $\sigma_2$ | $(1\;3)$ | Transposition |
| $\sigma_3$ | $(2\;3)$ | Transposition |
| $\sigma_4$ | $(1\;2\;3)$ | 3-cycle |
| $\sigma_5$ | $(1\;3\;2)$ | 3-cycle |

**Cayley table for $S_3$** (partial, using abbreviations):

```
Let: e = (), a = (123), b = (132), c = (12), d = (13), f = (23)

   ∘  │  e    a    b    c    d    f
  ────┼──────────────────────────────
   e  │  e    a    b    c    d    f
   a  │  a    b    e    f    c    d
   b  │  b    e    a    d    f    c
   c  │  c    d    f    e    a    b
   d  │  d    f    c    b    e    a
   f  │  f    c    d    a    b    e
```

Note: $a \circ c = f$ but $c \circ a = d$, so $S_3$ is **non-abelian**.

### 4.5 The Dihedral Group $D_n$

The symmetries of a regular $n$-gon. $|D_n| = 2n$.

**$D_3$ — Symmetries of an equilateral triangle:**

```
         1
        /  \          Rotations:  e, r, r²
       /    \         Reflections: s, rs, r²s
      /      \
     3────────2       r = rotation by 120°
                      s = reflection about axis through vertex 1
```

$D_3 \cong S_3$ (these are isomorphic groups!).

**$D_4$ — Symmetries of a square:**

```
     1 ─────── 2
     │         │      Rotations:  e, r, r², r³
     │    ●    │      Reflections: s, rs, r²s, r³s
     │         │
     4 ─────── 3      |D₄| = 8
```

### 4.6 Matrix Groups

**General Linear Group:** $GL_n(\mathbb{R}) = \{A \in M_n(\mathbb{R}) : \det(A) \neq 0\}$ under matrix multiplication.

- Identity: $I_n$
- Inverse: $A^{-1}$ (exists since $\det(A) \neq 0$)
- Non-abelian for $n \geq 2$

**Special Linear Group:** $SL_n(\mathbb{R}) = \{A \in GL_n(\mathbb{R}) : \det(A) = 1\}$

### 4.7 The Trivial Group

$G = \{e\}$ with $e \ast e = e$. This is the smallest possible group.

### 4.8 The Klein Four-Group $V_4$

$V_4 = \{e, a, b, c\}$ where every non-identity element has order 2:

```
  ∗  │  e   a   b   c
 ────┼────────────────
  e  │  e   a   b   c
  a  │  a   e   c   b
  b  │  b   c   e   a
  c  │  c   b   a   e
```

- Abelian, $|V_4| = 4$
- $V_4 \cong \mathbb{Z}_2 \times \mathbb{Z}_2$
- Not cyclic (no single generator)!

---

## 5. Non-Examples (Why Axioms Fail)

| Structure | Which axiom fails? |
|-----------|-------------------|
| $(\mathbb{Z}, -)$ | Not associative: $(1-2)-3 \neq 1-(2-3)$ |
| $(\mathbb{N}, +)$ | No inverses (no negatives) |
| $(\mathbb{Z}, \cdot)$ | No inverses ($2$ has no integer multiplicative inverse) |
| $(M_2(\mathbb{R}), \cdot)$ | No inverses (singular matrices) |
| $(\mathbb{R}, \cdot)$ | No inverse for $0$ |

---

## 6. Reading Cayley Tables

A Cayley table completely determines a finite group. Key observations:

```
Properties visible in a Cayley table:
┌──────────────────────────────────────────────┐
│ 1. Each row is a permutation of G            │
│    (every element appears exactly once)       │
│                                               │
│ 2. Each column is a permutation of G          │
│    (Latin square property)                    │
│                                               │
│ 3. The identity row/column reproduces G       │
│                                               │
│ 4. Symmetric table ⟺ abelian group           │
│                                               │
│ 5. a² = e visible on the diagonal             │
└──────────────────────────────────────────────┘
```

---

## 7. Notation Conventions

| Context | Operation | Identity | Inverse of $a$ | $a \ast a \ast \cdots \ast a$ ($n$ times) |
|---------|-----------|----------|-----------------|------------------------------------------|
| **Multiplicative** | $ab$ or $a \cdot b$ | $1$ or $e$ | $a^{-1}$ | $a^n$ |
| **Additive** | $a + b$ | $0$ | $-a$ | $na$ |

> Additive notation is typically reserved for **abelian** groups.

---

## 8. Real-World Applications

| Application | Group Used |
|-------------|-----------|
| Cryptography (RSA) | $(\mathbb{Z}_n^\times, \cdot)$ |
| Physics (symmetries) | $SO(3)$, Lorentz group |
| Chemistry (molecular symmetry) | Point groups ($C_n$, $D_n$, etc.) |
| Rubik's cube | Subgroup of $S_{48}$ |
| Music theory | $(\mathbb{Z}_{12}, +)$ — pitch classes |
| Error-correcting codes | Abelian groups |

---

## Summary Table

| Concept | Definition / Key Fact |
|---------|----------------------|
| Binary operation | $\ast: S \times S \to S$ |
| Group | $(G, \ast)$ satisfying closure, associativity, identity, inverses |
| Abelian group | Group with $ab = ba$ for all $a, b$ |
| Order of a group | $\|G\|$ = number of elements |
| Cayley table | Multiplication table; Latin square |
| $(\mathbb{Z}_n, +)$ | Cyclic group of order $n$ |
| $S_n$ | Symmetric group, order $n!$, non-abelian for $n \geq 3$ |
| $D_n$ | Dihedral group, order $2n$ |
| $GL_n(\mathbb{R})$ | Invertible $n \times n$ matrices |
| $V_4$ | Klein four-group, $\cong \mathbb{Z}_2 \times \mathbb{Z}_2$ |

---

## Quick Revision Questions

1. **State all four group axioms.** Which axiom distinguishes a group from a monoid?

2. **Prove** that the identity element in a group is unique.

3. **Determine** whether $(\mathbb{Q}, \cdot)$ is a group. If not, what fails?

4. **Construct** the Cayley table for $(\mathbb{Z}_3, +)$ and verify all group axioms from it.

5. **Show** that $S_3$ is non-abelian by finding two elements that do not commute.

6. **Explain** why the set of all $2 \times 2$ matrices under multiplication does **not** form a group, but $GL_2(\mathbb{R})$ does.

---

[← Back to README](../README.md) | [Next: Order of an Element →](02-order-of-element.md)
