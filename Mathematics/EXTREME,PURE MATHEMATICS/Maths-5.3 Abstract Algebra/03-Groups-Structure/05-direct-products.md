# Chapter 3.5 — Direct Products

[← Previous: Simple Groups](04-simple-groups.md) | [Back to README](../README.md) | [Next: Group Actions →](../04-Groups-Actions-Sylow/01-group-actions.md)

---

## Overview

**Direct products** let us build larger groups from smaller ones. Given groups $G$ and $H$, their direct product $G \times H$ is the "Cartesian product with component-wise operations". Understanding when a group decomposes as a direct product is key to the classification of finitely generated abelian groups.

---

## 1. External Direct Product

**Definition.** The **external direct product** of groups $G$ and $H$ is:

$$G \times H = \{(g, h) : g \in G, h \in H\}$$

with operation $(g_1, h_1)(g_2, h_2) = (g_1 g_2, h_1 h_2)$.

| Property | Value |
|----------|-------|
| Identity | $(e_G, e_H)$ |
| Inverse | $(g, h)^{-1} = (g^{-1}, h^{-1})$ |
| Order | $\|G \times H\| = \|G\| \cdot \|H\|$ |
| Abelian? | G × H abelian $\iff$ both $G$ and $H$ are abelian |

### Order of Elements

$$|(g, h)| = \text{lcm}(|g|, |h|)$$

### Example: $\mathbb{Z}_2 \times \mathbb{Z}_3$

| $(a,b)$ | $\|(a,b)\|$ |
|---------|-----------|
| $(0,0)$ | 1 |
| $(1,0)$ | 2 |
| $(0,1)$ | 3 |
| $(0,2)$ | 3 |
| $(1,1)$ | lcm(2,3) = 6 |
| $(1,2)$ | lcm(2,3) = 6 |

Since $(1,1)$ has order 6 = $|G \times H|$, this group is **cyclic**:

$$\mathbb{Z}_2 \times \mathbb{Z}_3 \cong \mathbb{Z}_6$$

---

## 2. When Is $\mathbb{Z}_m \times \mathbb{Z}_n$ Cyclic?

**Theorem 3.18.** $\mathbb{Z}_m \times \mathbb{Z}_n \cong \mathbb{Z}_{mn}$ if and only if $\gcd(m, n) = 1$.

```
  Z_m × Z_n  ≅  Z_{mn}   ⟺   gcd(m,n) = 1
  
  Examples:
  Z₂ × Z₃ ≅ Z₆          gcd(2,3) = 1  ✓
  Z₂ × Z₄ ≇ Z₈          gcd(2,4) = 2  ✗
  Z₃ × Z₅ ≅ Z₁₅         gcd(3,5) = 1  ✓
  Z₂ × Z₂ ≇ Z₄          gcd(2,2) = 2  ✗  (this is V₄)
  
  General: Z_{n₁} × Z_{n₂} × ··· × Z_{nₖ} ≅ Z_{n₁n₂···nₖ}
           ⟺ the nᵢ are pairwise coprime
```

---

## 3. Internal Direct Product

**Definition.** $G$ is the **internal direct product** of normal subgroups $H$ and $K$ (written $G = H \times K$) if:
1. $H \trianglelefteq G$ and $K \trianglelefteq G$
2. $H \cap K = \{e\}$
3. $G = HK$

**Theorem 3.19.** If $G = H \times K$ (internally), then $G \cong H \times K$ (externally).

```
  Internal vs External Direct Product:
  
  EXTERNAL:                    INTERNAL:
  G × H                        G = H × K
  = {(g,h)}                    H, K ◁ G
  component-wise                H ∩ K = {e}
  operation                     G = HK
  
  ┌──────────────────────────────────────────┐
  │  These give ISOMORPHIC groups!           │
  │  Internal ⟺ External (up to ≅)         │
  └──────────────────────────────────────────┘
```

### Example: $V_4$ as Internal Direct Product

$V_4 = \{e, a, b, c\}$ where $a^2 = b^2 = c^2 = e$, $ab = c$.

Let $H = \{e, a\} \cong \mathbb{Z}_2$ and $K = \{e, b\} \cong \mathbb{Z}_2$.

1. $H, K \trianglelefteq V_4$ (abelian group) ✓
2. $H \cap K = \{e\}$ ✓
3. $HK = \{e, a, b, ab\} = \{e, a, b, c\} = V_4$ ✓

So $V_4 = H \times K \cong \mathbb{Z}_2 \times \mathbb{Z}_2$. ✓

---

## 4. Fundamental Theorem of Finitely Generated Abelian Groups

**Theorem 3.20 (FTFGAG).** Every finitely generated abelian group is isomorphic to:

$$\mathbb{Z}^r \times \mathbb{Z}_{n_1} \times \mathbb{Z}_{n_2} \times \cdots \times \mathbb{Z}_{n_k}$$

where $r \geq 0$ and $n_1 \mid n_2 \mid \cdots \mid n_k$ (each $n_i \geq 2$).

The number $r$ is the **rank** (or **free rank**), and the $n_i$ are the **invariant factors**.

**Equivalently (primary decomposition):**

$$G \cong \mathbb{Z}^r \times \mathbb{Z}_{p_1^{a_1}} \times \mathbb{Z}_{p_2^{a_2}} \times \cdots \times \mathbb{Z}_{p_m^{a_m}}$$

where the $p_i^{a_i}$ are the **elementary divisors**.

### Example: Abelian Groups of Order 36

$36 = 2^2 \cdot 3^2$

By the classification:

| Invariant factors | Elementary divisors | Group |
|-------------------|-------------------|-------|
| $36$ | $4, 9$ | $\mathbb{Z}_4 \times \mathbb{Z}_9 \cong \mathbb{Z}_{36}$ |
| $2, 18$ | $2, 2, 9$ | $\mathbb{Z}_2 \times \mathbb{Z}_{18}$ |
| $6, 6$ | $2, 3, 2, 3$ | $\mathbb{Z}_6 \times \mathbb{Z}_6 \cong \mathbb{Z}_2 \times \mathbb{Z}_3 \times \mathbb{Z}_2 \times \mathbb{Z}_3$ |
| $2, 2, 9$ | $2, 2, 9$ | $\mathbb{Z}_2 \times \mathbb{Z}_2 \times \mathbb{Z}_9$ |

Wait, let me be more careful. The elementary divisors are prime powers:

From $2^2$: possibilities are $\{4\}$ or $\{2, 2\}$
From $3^2$: possibilities are $\{9\}$ or $\{3, 3\}$

| $2$-part | $3$-part | Group | Invariant factors |
|----------|----------|-------|-------------------|
| $\mathbb{Z}_4$ | $\mathbb{Z}_9$ | $\mathbb{Z}_{36}$ | $36$ |
| $\mathbb{Z}_4$ | $\mathbb{Z}_3 \times \mathbb{Z}_3$ | $\mathbb{Z}_4 \times \mathbb{Z}_3 \times \mathbb{Z}_3 \cong \mathbb{Z}_{12} \times \mathbb{Z}_3$ | $3, 12$ |
| $\mathbb{Z}_2 \times \mathbb{Z}_2$ | $\mathbb{Z}_9$ | $\mathbb{Z}_2 \times \mathbb{Z}_{18}$ | $2, 18$ |
| $\mathbb{Z}_2 \times \mathbb{Z}_2$ | $\mathbb{Z}_3 \times \mathbb{Z}_3$ | $\mathbb{Z}_6 \times \mathbb{Z}_6$ | $6, 6$ |

**4 abelian groups of order 36** (up to isomorphism).

```
  Counting abelian groups of order n:
  
  n = p₁^a₁ · p₂^a₂ · ··· · pₖ^aₖ
  
  Number of abelian groups = ∏ᵢ p(aᵢ)
  
  where p(a) = number of partitions of a
  
  p(1) = 1:  {1}
  p(2) = 2:  {2}, {1,1}
  p(3) = 3:  {3}, {2,1}, {1,1,1}
  p(4) = 5:  {4}, {3,1}, {2,2}, {2,1,1}, {1,1,1,1}
```

---

## 5. Recognising Direct Products

**Theorem 3.21.** A finite group $G$ with normal subgroups $H, K$ satisfying $H \cap K = \{e\}$ and $|H| \cdot |K| = |G|$ is isomorphic to $H \times K$.

### Checklist for Recognising $G \cong H \times K$:

```
  ┌─────────────────────────────────────────────┐
  │  To show G ≅ H × K:                         │
  │                                              │
  │  □  H ◁ G                                   │
  │  □  K ◁ G                                   │
  │  □  H ∩ K = {e}                             │
  │  □  |H| · |K| = |G|  (or HK = G)           │
  │                                              │
  │  All four ⟹ G ≅ H × K                      │
  └─────────────────────────────────────────────┘
```

---

## 6. Direct Product of Multiple Groups

Extends naturally: $G_1 \times G_2 \times \cdots \times G_n$

Elements: $(g_1, g_2, \ldots, g_n)$, operation component-wise.

$(g_1, \ldots, g_n) \cdot (h_1, \ldots, h_n) = (g_1 h_1, \ldots, g_n h_n)$

### Subgroups of Direct Products

The subgroups of $G \times H$ include:
- $G \times \{e_H\} \cong G$ and $\{e_G\} \times H \cong H$
- Diagonal subgroups (if $G \cong H$): $\Delta = \{(g, g) : g \in G\}$
- But **not all** subgroups have the form $A \times B$!

---

## 7. Semidirect Products (Preview)

When $G$ does **not** decompose as a direct product, but has $N \trianglelefteq G$ and $H \leq G$ with $NH = G$ and $N \cap H = \{e\}$, we get a **semidirect product** $G \cong N \rtimes H$.

| Structure | Conditions | Notation |
|-----------|-----------|----------|
| Direct product | Both $N, H$ normal | $N \times H$ |
| Semidirect product | Only $N$ normal | $N \rtimes H$ |

**Example:** $S_3 \cong A_3 \rtimes \langle(12)\rangle$, but $S_3 \not\cong \mathbb{Z}_3 \times \mathbb{Z}_2$ since $S_3$ is non-abelian.

---

## 8. Real-World Applications

| Application | Direct Product |
|------------|---------------|
| Chinese Remainder Theorem | $\mathbb{Z}_{mn} \cong \mathbb{Z}_m \times \mathbb{Z}_n$ when $\gcd(m,n) = 1$ |
| Multi-dimensional symmetry | $D_n \times D_m$ for independent symmetries |
| Parallel computation | Independent components |
| Product codes (coding theory) | $C_1 \times C_2$ |

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| $G \times H$ | $= \{(g,h)\}$, component-wise operation |
| $\|(g,h)\|$ | $= \text{lcm}(\|g\|, \|h\|)$ |
| $\mathbb{Z}_m \times \mathbb{Z}_n \cong \mathbb{Z}_{mn}$ | $\iff \gcd(m,n) = 1$ |
| Internal $= $ external | Same up to isomorphism |
| FTFGAG | Every f.g. abelian group $\cong \mathbb{Z}^r \times \prod \mathbb{Z}_{n_i}$ |
| Counting | Abelian groups of order $n = \prod p_i^{a_i}$: count $= \prod p(a_i)$ |
| Recognition | $H, K \trianglelefteq G$, $H \cap K = \{e\}$, $HK = G$ |

---

## Quick Revision Questions

1. **Show** that $\mathbb{Z}_2 \times \mathbb{Z}_3 \cong \mathbb{Z}_6$ by finding a generator.

2. **Prove** that $\mathbb{Z}_2 \times \mathbb{Z}_2 \not\cong \mathbb{Z}_4$.

3. **List** all abelian groups of order 72, up to isomorphism.

4. **Verify** that $V_4$ is an internal direct product of two copies of $\mathbb{Z}_2$.

5. **State** the Fundamental Theorem of Finitely Generated Abelian Groups and apply it to groups of order 100.

6. **Explain** the difference between direct and semidirect products, giving an example of each.

---

[← Previous: Simple Groups](04-simple-groups.md) | [Back to README](../README.md) | [Next: Group Actions →](../04-Groups-Actions-Sylow/01-group-actions.md)
