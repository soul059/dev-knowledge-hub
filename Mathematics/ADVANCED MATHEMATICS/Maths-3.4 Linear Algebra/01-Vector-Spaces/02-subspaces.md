# 1.2 Subspaces

[← Previous: Definition and Examples](01-definition-and-examples.md) · [Back to README](../README.md) · [Next: Linear Combination →](03-linear-combination.md)

---

## Chapter Overview

Not every subset of a vector space is itself a vector space. A **subspace** is a subset that *is* a vector space under the same operations. This chapter gives the definition, the efficient **subspace test**, and many worked examples.

---

## 1. Definition

**Definition.** Let $V$ be a vector space over a field $F$. A non-empty subset $W \subseteq V$ is called a **subspace** of $V$ if $W$ is itself a vector space over $F$ under the same addition and scalar multiplication.

```
  ┌───────────────────────────────┐
  │            V                  │
  │    ┌───────────────┐          │
  │    │       W       │  ◄─ subspace │
  │    │   (closed     │          │
  │    │   under + , · │          │
  │    │   contains 0) │          │
  │    └───────────────┘          │
  └───────────────────────────────┘
```

---

## 2. The Subspace Test (The Efficient Way)

Checking all 10 axioms is tedious. Since $W$ inherits most axioms from $V$, we only need:

> **Subspace Test.** A non-empty subset $W$ of a vector space $V$ is a subspace if and only if:
>
> 1. **Zero vector:** $\mathbf{0} \in W$
> 2. **Closed under addition:** $\mathbf{u}, \mathbf{v} \in W \Rightarrow \mathbf{u} + \mathbf{v} \in W$
> 3. **Closed under scalar multiplication:** $\mathbf{u} \in W,\; c \in F \Rightarrow c\mathbf{u} \in W$

Equivalently (combining 2 & 3 into one check):

> $W \neq \varnothing$ and $\forall\; \mathbf{u}, \mathbf{v} \in W,\; \forall\; c \in F:\; \mathbf{u} + c\mathbf{v} \in W$

**Why this suffices:**
- Commutativity, associativity, distributivity are inherited from $V$.
- Condition 3 with $c = -1$ gives additive inverses.
- Condition 1 ensures non-emptiness and provides the identity.

---

## 3. Flowchart for the Subspace Test

```
  Start: Is W a subspace of V?
  │
  ▼
  ┌─────────────────┐    NO
  │  Is 0 ∈ W ?     ├────────▶  NOT a subspace
  └────────┬────────┘
           │ YES
           ▼
  ┌─────────────────────────┐    NO
  │ u, v ∈ W ⇒ u + v ∈ W ? ├────────▶  NOT a subspace
  └────────┬────────────────┘
           │ YES
           ▼
  ┌─────────────────────────────┐    NO
  │ u ∈ W, c ∈ F ⇒ c·u ∈ W ?  ├────────▶  NOT a subspace
  └────────┬────────────────────┘
           │ YES
           ▼
      W IS a subspace ✓
```

---

## 4. Examples of Subspaces

### Example 1: Lines Through the Origin in $\mathbb{R}^2$

$W = \{(x, y) \in \mathbb{R}^2 \mid y = mx\}$ for a fixed slope $m$.

```
        y
        ▲
       /│          W = line y = 2x
      / │         /
     /  │        /
    /   │       /
   /    │      /
  ──────┼─────/──────▶ x
        │    /
        │   /
        │  /
```

**Check:**
1. $(0, 0) \in W$ ✓ (since $0 = m \cdot 0$)
2. If $(x_1, mx_1), (x_2, mx_2) \in W$, then $(x_1+x_2, m(x_1+x_2)) \in W$ ✓
3. $c(x, mx) = (cx, m(cx)) \in W$ ✓

### Example 2: Planes Through the Origin in $\mathbb{R}^3$

$W = \{(x, y, z) \mid ax + by + cz = 0\}$ — a homogeneous linear equation.

**Check:**
1. $(0,0,0)$ satisfies $a(0)+b(0)+c(0)=0$ ✓
2. If $a x_1 + b y_1 + c z_1 = 0$ and $a x_2 + b y_2 + c z_2 = 0$, then $a(x_1+x_2) + b(y_1+y_2) + c(z_1+z_2) = 0$ ✓
3. $a(kx) + b(ky) + c(kz) = k(ax+by+cz) = 0$ ✓

### Example 3: The Null Space of a Matrix

For an $m \times n$ matrix $A$:

$$\text{Null}(A) = \{\mathbf{x} \in \mathbb{R}^n \mid A\mathbf{x} = \mathbf{0}\}$$

This is always a subspace of $\mathbb{R}^n$.

**Proof:**
1. $A\mathbf{0} = \mathbf{0}$ ✓
2. $A(\mathbf{u}+\mathbf{v}) = A\mathbf{u}+A\mathbf{v} = \mathbf{0}+\mathbf{0} = \mathbf{0}$ ✓
3. $A(c\mathbf{u}) = cA\mathbf{u} = c\mathbf{0} = \mathbf{0}$ ✓

### Example 4: Symmetric Matrices

$W = \{A \in M_{n \times n}(\mathbb{R}) \mid A^T = A\}$

1. The zero matrix is symmetric ✓
2. $(A+B)^T = A^T + B^T = A + B$ ✓
3. $(cA)^T = cA^T = cA$ ✓

### Example 5: $P_n$ is a Subspace of All Polynomials

$P_n \subset P(\mathbb{R})$ (polynomials of degree $\leq n$) is a subspace:
- The zero polynomial has degree $-\infty \leq n$ ✓
- Sum of two degree-$\leq n$ polynomials has degree $\leq n$ ✓
- Scalar multiple preserves degree ✓

---

## 5. Non-Examples of Subspaces

| Set | Why It Fails |
|-----|-------------|
| $\{(x,y) \mid x + y = 1\}$ | Does not contain $\mathbf{0}$ (since $0+0 \neq 1$) |
| $\{(x,y) \mid xy \geq 0\}$ | Not closed under addition: $(1,1)+(-2,0)=(-1,1)$, and $(-1)(1)<0$ |
| Upper triangular matrices with all diagonal entries = 1 | Not closed under scalar multiplication |
| Polynomials of degree *exactly* 3 | Not closed under addition (leading terms can cancel) |
| Invertible $n \times n$ matrices | The zero matrix is not invertible |

---

## 6. Intersections and Sums of Subspaces

### Intersection

> **Theorem.** If $W_1$ and $W_2$ are subspaces of $V$, then $W_1 \cap W_2$ is also a subspace of $V$.

**Proof sketch:** $\mathbf{0} \in W_1 \cap W_2$. If $\mathbf{u}, \mathbf{v} \in W_1 \cap W_2$, then both are in $W_1$ (so $\mathbf{u}+\mathbf{v} \in W_1$) and both are in $W_2$ (so $\mathbf{u}+\mathbf{v} \in W_2$). Similarly for scalar multiples.

```
  ┌──────────────────────────┐
  │           V              │
  │   ┌──────┼──────┐        │
  │   │  W₁  │  W₂  │        │
  │   │      │      │        │
  │   │   W₁∩W₂    │        │
  │   │  (subspace) │        │
  │   └──────┼──────┘        │
  └──────────────────────────┘
```

### Sum

$W_1 + W_2 = \{\mathbf{w}_1 + \mathbf{w}_2 \mid \mathbf{w}_1 \in W_1,\; \mathbf{w}_2 \in W_2\}$

This is also a subspace, and it is the **smallest** subspace containing both $W_1$ and $W_2$.

### Direct Sum

$V = W_1 \oplus W_2$ if $V = W_1 + W_2$ and $W_1 \cap W_2 = \{\mathbf{0}\}$.

In a direct sum, every $\mathbf{v} \in V$ can be written **uniquely** as $\mathbf{v} = \mathbf{w}_1 + \mathbf{w}_2$.

> **Warning:** $W_1 \cup W_2$ is generally **NOT** a subspace (unless one is contained in the other).

---

## 7. The Solution Space of a Homogeneous System

A homogeneous system $A\mathbf{x} = \mathbf{0}$ always has the trivial solution $\mathbf{x} = \mathbf{0}$. The set of *all* solutions is a subspace of $\mathbb{R}^n$ (this is the null space $\text{Null}(A)$).

A non-homogeneous system $A\mathbf{x} = \mathbf{b}$ ($\mathbf{b} \neq \mathbf{0}$) has a solution set that is **not** a subspace (it doesn't contain $\mathbf{0}$ unless $\mathbf{b} = \mathbf{0}$).

```
  Solution structure:
  
  General solution = Particular solution + Null-space solution
  
       x = x_p + x_h
  
  where  A·x_p = b   and   A·x_h = 0
```

---

## 8. Design Example — Verifying a Subspace Step by Step

**Problem:** Is $W = \{(a, b, c) \in \mathbb{R}^3 \mid a - 2b + c = 0\}$ a subspace of $\mathbb{R}^3$?

**Step 1:** Check $\mathbf{0} \in W$.  
$0 - 2(0) + 0 = 0$ ✓

**Step 2:** Take arbitrary $\mathbf{u} = (a_1, b_1, c_1),\; \mathbf{v} = (a_2, b_2, c_2) \in W$.  
So $a_1 - 2b_1 + c_1 = 0$ and $a_2 - 2b_2 + c_2 = 0$.

$\mathbf{u} + \mathbf{v} = (a_1+a_2, b_1+b_2, c_1+c_2)$

$(a_1+a_2) - 2(b_1+b_2) + (c_1+c_2) = (a_1-2b_1+c_1) + (a_2-2b_2+c_2) = 0 + 0 = 0$ ✓

**Step 3:** $k \cdot \mathbf{u} = (ka_1, kb_1, kc_1)$

$ka_1 - 2(kb_1) + kc_1 = k(a_1 - 2b_1 + c_1) = k \cdot 0 = 0$ ✓

**Conclusion:** $W$ is a subspace of $\mathbb{R}^3$. ✓

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Subspace | A subset that is itself a vector space under inherited operations |
| Subspace test (3 checks) | (1) $\mathbf{0} \in W$, (2) closed under $+$, (3) closed under $\cdot$ |
| Null space $\text{Null}(A)$ | Always a subspace of $\mathbb{R}^n$ |
| Intersection $W_1 \cap W_2$ | Always a subspace |
| Sum $W_1 + W_2$ | Always a subspace |
| Union $W_1 \cup W_2$ | Generally **not** a subspace |
| Direct sum $W_1 \oplus W_2$ | Sum with trivial intersection; unique decomposition |
| Non-homogeneous solutions | **Not** a subspace (shifted by a particular solution) |

---

## Quick Revision Questions

1. State the three conditions of the subspace test. Why don't we need to check all 10 vector space axioms?

2. Prove that the set of all $n \times n$ **diagonal** matrices is a subspace of $M_{n \times n}(\mathbb{R})$.

3. Is $W = \{(x, y, z) \in \mathbb{R}^3 \mid x^2 + y^2 + z^2 \leq 1\}$ a subspace of $\mathbb{R}^3$? Justify.

4. If $W_1$ and $W_2$ are subspaces of $V$, when is $W_1 \cup W_2$ also a subspace?

5. Give an example of two subspaces of $\mathbb{R}^3$ whose direct sum equals $\mathbb{R}^3$.

6. The solution set of $x + 2y - z = 5$ in $\mathbb{R}^3$ — is it a subspace? Explain.

---

[← Previous: Definition and Examples](01-definition-and-examples.md) · [Back to README](../README.md) · [Next: Linear Combination →](03-linear-combination.md)
