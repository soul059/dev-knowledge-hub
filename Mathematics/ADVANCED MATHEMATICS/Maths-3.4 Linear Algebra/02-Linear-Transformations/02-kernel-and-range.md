# 2.2 Kernel and Range

[← Previous: Definition and Examples](01-definition-and-examples.md) · [Back to README](../README.md) · [Next: Rank-Nullity Theorem →](03-rank-nullity-theorem.md)

---

## Chapter Overview

Every linear transformation $T : V \to W$ has two fundamental subspaces associated with it: the **kernel** (what $T$ "kills") and the **range** (what $T$ "hits"). Understanding these tells us about injectivity, surjectivity, and the structure of solutions to $T(\mathbf{x}) = \mathbf{b}$.

---

## 1. Definitions

### Kernel (Null Space)

$$\ker(T) = \{\mathbf{v} \in V \mid T(\mathbf{v}) = \mathbf{0}\}$$

The kernel is the set of all vectors that $T$ maps to the zero vector.

### Range (Image)

$$\text{Im}(T) = \text{Range}(T) = \{T(\mathbf{v}) \mid \mathbf{v} \in V\} = \{w \in W \mid \exists\; \mathbf{v} \in V : T(\mathbf{v}) = \mathbf{w}\}$$

The range is the set of all vectors in $W$ that are actually "reached" by $T$.

```
    Domain V                          Codomain W
  ┌──────────────────┐              ┌──────────────────┐
  │                  │              │                  │
  │   ┌────────┐     │     T       │   ┌────────┐     │
  │   │ ker(T) │─────┼────────────▶│   │   0    │     │
  │   │ (maps  │     │             │   └────────┘     │
  │   │ to 0)  │     │             │                  │
  │   └────────┘     │             │   ┌────────────┐ │
  │                  │             │   │  Im(T)     │ │
  │   other v ───────┼────────────▶│   │  (range)   │ │
  │                  │             │   └────────────┘ │
  └──────────────────┘              └──────────────────┘
```

---

## 2. Kernel and Range Are Subspaces

> **Theorem.** If $T : V \to W$ is linear, then:
> 1. $\ker(T)$ is a subspace of $V$
> 2. $\text{Im}(T)$ is a subspace of $W$

**Proof for ker(T):**
1. $T(\mathbf{0}_V) = \mathbf{0}_W$, so $\mathbf{0}_V \in \ker(T)$. ✓
2. If $\mathbf{u}, \mathbf{v} \in \ker(T)$: $T(\mathbf{u}+\mathbf{v}) = T(\mathbf{u})+T(\mathbf{v}) = \mathbf{0}+\mathbf{0} = \mathbf{0}$. ✓
3. If $\mathbf{u} \in \ker(T)$: $T(c\mathbf{u}) = cT(\mathbf{u}) = c\mathbf{0} = \mathbf{0}$. ✓

**Proof for Im(T):**
1. $T(\mathbf{0}) = \mathbf{0} \in \text{Im}(T)$. ✓
2. If $\mathbf{w}_1 = T(\mathbf{v}_1), \mathbf{w}_2 = T(\mathbf{v}_2) \in \text{Im}(T)$: $\mathbf{w}_1+\mathbf{w}_2 = T(\mathbf{v}_1+\mathbf{v}_2) \in \text{Im}(T)$. ✓
3. $c\mathbf{w}_1 = cT(\mathbf{v}_1) = T(c\mathbf{v}_1) \in \text{Im}(T)$. ✓

---

## 3. Kernel ↔ Injectivity, Range ↔ Surjectivity

| Property | Condition | Meaning |
|----------|-----------|---------|
| $T$ is **injective** (one-to-one) | $\ker(T) = \{\mathbf{0}\}$ | Different inputs give different outputs |
| $T$ is **surjective** (onto) | $\text{Im}(T) = W$ | Every $\mathbf{w} \in W$ is hit |
| $T$ is **bijective** (isomorphism) | Both conditions | One-to-one AND onto |

> **Theorem.** $T$ is injective $\iff$ $\ker(T) = \{\mathbf{0}\}$.

**Proof (⇒):** If $T$ is injective and $T(\mathbf{v}) = \mathbf{0} = T(\mathbf{0})$, then $\mathbf{v} = \mathbf{0}$.

**Proof (⇐):** If $\ker(T) = \{\mathbf{0}\}$ and $T(\mathbf{u}) = T(\mathbf{v})$, then $T(\mathbf{u}-\mathbf{v}) = \mathbf{0}$, so $\mathbf{u}-\mathbf{v} \in \ker(T) = \{\mathbf{0}\}$, hence $\mathbf{u} = \mathbf{v}$.

---

## 4. Computing the Kernel (for Matrix Transformations)

For $T(\mathbf{x}) = A\mathbf{x}$, $\ker(T) = \text{Null}(A)$.

**Algorithm:** Solve $A\mathbf{x} = \mathbf{0}$ by row-reducing $A$.

### Worked Example

$$A = \begin{pmatrix} 1 & 2 & 1 \\ 2 & 4 & 3 \\ 3 & 6 & 4 \end{pmatrix}$$

```
  ┌              ┐        ┌              ┐        ┌              ┐
  │ 1   2   1    │  R₂-2R₁│ 1   2   1    │  R₃-R₂ │ 1   2   1    │
  │ 2   4   3    │ ──────▶│ 0   0   1    │ ──────▶│ 0   0   1    │
  │ 3   6   4    │  R₃-3R₁│ 0   0   1    │        │ 0   0   0    │
  └              ┘        └              ┘        └              ┘
```

Back-substitute: $x_3 = 0$, $x_1 = -2x_2$, $x_2$ free.

$$\ker(T) = \text{span}\left\{\begin{pmatrix}-2\\1\\0\end{pmatrix}\right\}$$

$\dim(\ker(T)) = 1$ (one free variable).

---

## 5. Computing the Range (for Matrix Transformations)

$\text{Im}(T) = \text{Col}(A)$ = span of the columns of $A$.

**Algorithm:** Row-reduce $A$, identify pivot columns in the *original* $A$.

From the example above, pivots are in columns 1 and 3:

$$\text{Im}(T) = \text{span}\left\{\begin{pmatrix}1\\2\\3\end{pmatrix}, \begin{pmatrix}1\\3\\4\end{pmatrix}\right\}$$

$\dim(\text{Im}(T)) = 2$ (two pivot columns).

---

## 6. Kernel and Range for Non-Matrix Transformations

### Example: Differentiation $D : P_3 \to P_2$

$D(a + bx + cx^2 + dx^3) = b + 2cx + 3dx^2$

**Kernel:** $D(p) = 0 \iff p = a$ (constant polynomials). $\ker(D) = P_0 = \text{span}\{1\}$, $\dim = 1$.

**Range:** $b + 2cx + 3dx^2$ — any polynomial of degree $\leq 2$. $\text{Im}(D) = P_2$, $\dim = 3$.

Check: $\dim(\ker(D)) + \dim(\text{Im}(D)) = 1 + 3 = 4 = \dim(P_3)$. ✓ (Rank-Nullity!)

### Example: Trace $\text{tr} : M_{2 \times 2} \to \mathbb{R}$

**Kernel:** $\text{tr}(A) = 0 \iff a_{11} + a_{22} = 0$.

$$\ker(\text{tr}) = \left\{\begin{pmatrix}a & b \\ c & -a\end{pmatrix}\right\} = \text{span}\left\{\begin{pmatrix}1&0\\0&-1\end{pmatrix}, \begin{pmatrix}0&1\\0&0\end{pmatrix}, \begin{pmatrix}0&0\\1&0\end{pmatrix}\right\}$$

$\dim(\ker) = 3$. $\text{Im}(\text{tr}) = \mathbb{R}$, $\dim = 1$. Rank-Nullity: $3 + 1 = 4 = \dim(M_{2\times 2})$. ✓

---

## 7. Visual Summary

```
  ┌──────────────────────────────────────────────────────┐
  │                                                      │
  │          T : V ────────────────────▶ W               │
  │                                                      │
  │    ┌─────────────┐            ┌─────────────┐        │
  │    │   ker(T)    │ ──T──▶    │     {0}     │        │
  │    │ (subspace   │            │             │        │
  │    │  of V)      │            │             │        │
  │    │ nullity =   │            │             │        │
  │    │ dim(ker(T)) │            │             │        │
  │    └─────────────┘            │             │        │
  │                               │   ┌─────────┴──┐    │
  │    ┌─────────────┐            │   │   Im(T)    │    │
  │    │ rest of V   │ ──T──▶    │   │ (subspace  │    │
  │    │             │            │   │  of W)     │    │
  │    │             │            │   │ rank =     │    │
  │    │             │            │   │ dim(Im(T)) │    │
  │    └─────────────┘            │   └────────────┘    │
  │                               └────────────────┘    │
  └──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| $\ker(T)$ | $\{\mathbf{v} \mid T(\mathbf{v})=\mathbf{0}\}$; subspace of $V$ |
| $\text{Im}(T)$ | $\{T(\mathbf{v}) \mid \mathbf{v} \in V\}$; subspace of $W$ |
| Injective | $\iff \ker(T) = \{\mathbf{0}\}$ |
| Surjective | $\iff \text{Im}(T) = W$ |
| For $T(\mathbf{x})=A\mathbf{x}$ | $\ker(T) = \text{Null}(A)$, $\text{Im}(T) = \text{Col}(A)$ |
| Nullity | $\dim(\ker(T))$ = number of free variables |
| Rank | $\dim(\text{Im}(T))$ = number of pivot columns |

---

## Quick Revision Questions

1. Find the kernel and range of $T(x, y) = (x + y, x - y, 2x)$.

2. Is the linear transformation $T(x,y,z) = (x+y, y+z)$ injective? Surjective?

3. Prove that $\ker(T)$ is a subspace of $V$.

4. Find a basis for the kernel of $A = \begin{pmatrix}1&1&1\\1&2&3\\2&3&4\end{pmatrix}$.

5. If $T : \mathbb{R}^5 \to \mathbb{R}^3$ is linear and $\dim(\ker(T)) = 3$, what is $\dim(\text{Im}(T))$?

6. Give an example of a linear transformation that is surjective but not injective.

---

[← Previous: Definition and Examples](01-definition-and-examples.md) · [Back to README](../README.md) · [Next: Rank-Nullity Theorem →](03-rank-nullity-theorem.md)
