# Chapter 9.2 — The Fundamental Group

[← Previous: Homotopy](01-homotopy.md) | [Back to README](../README.md) | [Next: Covering Spaces →](03-covering-spaces.md)

---

## 1. Overview

The **fundamental group** $\pi_1(X, x_0)$ captures the "loop structure" of a space — an algebraic invariant that detects holes. It is the first and most important of the homotopy groups and is the entry point to algebraic topology.

---

## 2. Definition

### 2.1 Loops and the Loop Space

> **Def 2.1.** A **loop** based at $x_0 \in X$ is a path $\gamma : [0,1] \to X$ with $\gamma(0) = \gamma(1) = x_0$.

> **Def 2.2.** Two loops $\gamma, \delta$ based at $x_0$ are **equivalent** if they are path homotopic (homotopic rel $\{0,1\}$). Write $[\gamma]$ for the homotopy class.

### 2.2 The Group

> **Def 2.3.** The **fundamental group** is
> $$\pi_1(X, x_0) = \{[\gamma] : \gamma \text{ is a loop based at } x_0\}$$
> with the operation of **concatenation**:
> $$[\gamma] \cdot [\delta] = [\gamma * \delta]$$
> where
> $$(\gamma * \delta)(t) = \begin{cases} \gamma(2t) & 0 \leq t \leq 1/2 \\ \delta(2t-1) & 1/2 \leq t \leq 1 \end{cases}$$

```
Concatenation γ * δ:
        ╭── γ ──╮╭── δ ──╮
   x₀ ──┤        ├┤        ├── x₀
        ╰────────╯╰────────╯
   t:  0    →   1/2   →    1
   
   "Walk γ at double speed, then δ at double speed."
```

---

## 3. Group Axioms Verification

> **Thm 2.4.** $(\pi_1(X, x_0), \cdot)$ is a group.

| Axiom | Element | Proof idea |
|-------|---------|------------|
| **Identity** | $[e_{x_0}]$ (constant loop) | $\gamma * e \simeq \gamma$ via reparametrisation |
| **Inverse** | $[\bar{\gamma}]$ where $\bar{\gamma}(t) = \gamma(1-t)$ | $\gamma * \bar{\gamma} \simeq e_{x_0}$ (retrace and shrink) |
| **Associativity** | — | $(\gamma * \delta) * \eta \simeq \gamma * (\delta * \eta)$ via reparametrisation |
| **Closure** | — | Concatenation of loops is a loop; hom. class is well-defined |

---

## 4. Dependence on Basepoint

> **Thm 2.5.** If $X$ is path-connected and $\alpha$ is a path from $x_0$ to $x_1$, then the map
> $$\hat{\alpha} : \pi_1(X, x_0) \to \pi_1(X, x_1), \qquad [\gamma] \mapsto [\bar{\alpha} * \gamma * \alpha]$$
> is a group isomorphism.

So for path-connected spaces, $\pi_1$ is independent of basepoint **up to isomorphism**, and we write $\pi_1(X)$.

```
Change of basepoint:
                ╭── γ ──╮
   x₀ ──α──→ x₁──┤        ├── x₁ ──ᾱ──→ x₀
                ╰────────╯
   
   α̂([γ]) = [ᾱ * γ * α]   (conjugation by α)
```

**Note:** The isomorphism depends on the choice of path $\alpha$ — different paths give isomorphisms that differ by an inner automorphism.

---

## 5. Computations

| Space | $\pi_1$ | Reason |
|-------|---------|--------|
| $\mathbb{R}^n$ | $0$ | Contractible |
| $S^n$ ($n \geq 2$) | $0$ | Simply connected (van Kampen or covering space) |
| $S^1$ | $\mathbb{Z}$ | Winding number; covering space $\mathbb{R} \to S^1$ |
| Torus $T^2 = S^1 \times S^1$ | $\mathbb{Z} \times \mathbb{Z}$ | Product formula |
| Figure-eight $S^1 \vee S^1$ | $F_2$ (free group on 2 generators) | van Kampen |
| $\mathbb{R}P^2$ | $\mathbb{Z}/2\mathbb{Z}$ | Double cover $S^2 \to \mathbb{R}P^2$ |
| Klein bottle | $\langle a, b \mid abab^{-1}\rangle$ | Identification space |
| $\mathbb{R}^2 \setminus \{0\}$ | $\mathbb{Z}$ | Deformation retracts to $S^1$ |
| Genus-$g$ surface | $\langle a_1,b_1,\ldots,a_g,b_g \mid \prod [a_i,b_i] = 1\rangle$ | Classification of surfaces |

### 5.1 $\pi_1(S^1) \cong \mathbb{Z}$ — The Fundamental Example

The winding number map $\omega : \pi_1(S^1) \to \mathbb{Z}$ is an isomorphism. A loop that winds around the circle $n$ times represents $n \in \mathbb{Z}$.

```
Winding numbers in S¹:

  n = 0:  ╭──╮     n = 1:  ──→──    n = 2:  ──→──→──
          │  │              ↗   ↘             twice
          ╰──╯            ←─────            around
  (contractible)    (once around)
```

---

## 6. Functoriality

> **Thm 2.6.** $\pi_1$ is a **functor** from pointed topological spaces to groups:
> - A continuous map $f : (X, x_0) \to (Y, y_0)$ induces a group homomorphism $f_* : \pi_1(X, x_0) \to \pi_1(Y, y_0)$ via $f_*([\gamma]) = [f \circ \gamma]$.
> - $(g \circ f)_* = g_* \circ f_*$ and $(\mathrm{id})_* = \mathrm{id}$.

**Consequence:** If $f$ is a homeomorphism, then $f_*$ is an isomorphism. If $f$ is a homotopy equivalence, then $f_*$ is an isomorphism.

---

## 7. Applications

### 7.1 Brouwer Fixed-Point Theorem (Dimension 2)

> **Thm 2.7.** Every continuous $f : D^2 \to D^2$ has a fixed point.

*Proof:* Suppose $f(x) \neq x$ for all $x$. Define $r : D^2 \to S^1$ by sending $x$ to the point where the ray from $f(x)$ through $x$ hits $S^1$. Then $r$ is a retraction, so $r_* \circ i_* = \mathrm{id}$ on $\pi_1(S^1)$. But $\pi_1(D^2) = 0$ and $\pi_1(S^1) = \mathbb{Z}$: can't have $\mathbb{Z} \to 0 \to \mathbb{Z}$ equal identity. ∎

### 7.2 Fundamental Theorem of Algebra

Every non-constant polynomial $p(z) \in \mathbb{C}[z]$ has a root. (Proof via winding numbers.)

### 7.3 Borsuk–Ulam (Dimension 1)

For any continuous $f : S^1 \to \mathbb{R}$, there exists $x$ with $f(x) = f(-x)$.

---

## 8. Van Kampen's Theorem (Preview)

> **Thm 2.8 (Seifert–van Kampen).** If $X = U_1 \cup U_2$ with $U_1, U_2, U_1 \cap U_2$ open and path-connected, then
> $$\pi_1(X) \cong \pi_1(U_1) *_{\pi_1(U_1 \cap U_2)} \pi_1(U_2)$$
> (amalgamated free product).

This is the main computational tool for fundamental groups.

---

## 9. Summary Table

| Concept | Details |
|---------|---------|
| $\pi_1(X, x_0)$ | Homotopy classes of loops at $x_0$, with concatenation |
| Group structure | Identity = constant loop, inverse = reversed loop |
| Basepoint independence | $\pi_1(X, x_0) \cong \pi_1(X, x_1)$ if path-connected |
| Simply connected | $\pi_1 = 0$ |
| $\pi_1(S^1) = \mathbb{Z}$ | Winding number |
| Functoriality | $f_*([\gamma]) = [f \circ \gamma]$ |
| Applications | Brouwer FPT, FTA, classification of surfaces |

---

## 10. Quick Revision Questions

1. Define the fundamental group. Verify associativity, identity, and inverses.

2. Prove that $\pi_1$ is independent of basepoint (up to isomorphism) for path-connected spaces.

3. Compute $\pi_1(S^1)$ using the winding number / covering space approach.

4. State and prove the Brouwer Fixed-Point Theorem for the disk $D^2$.

5. Compute $\pi_1(T^2)$ and $\pi_1(S^1 \vee S^1)$.

6. What is van Kampen's theorem? Use it to compute $\pi_1(S^1 \vee S^1)$.

---

[← Previous: Homotopy](01-homotopy.md) | [Back to README](../README.md) | [Next: Covering Spaces →](03-covering-spaces.md)
