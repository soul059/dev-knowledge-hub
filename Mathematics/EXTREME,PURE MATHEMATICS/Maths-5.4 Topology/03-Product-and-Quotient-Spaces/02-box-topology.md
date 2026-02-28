# Chapter 3.2 — Box Topology

[← Previous: Product Topology](01-product-topology.md) | [Back to README](../README.md) | [Next: Quotient Topology →](03-quotient-topology.md)

---

## 1. Overview

The **box topology** is the "naïve" topology on an infinite product — allow all coordinates to be independently constrained. While natural-seeming, it has pathological properties that make it unsuitable for most purposes, which is why the **product topology** is preferred.

---

## 2. Definition

> **Def 2.1 (Box Topology).** Let $\{(X_\alpha, \mathcal{T}_\alpha)\}_{\alpha \in A}$ be a family of topological spaces. The **box topology** on $\prod_\alpha X_\alpha$ has basis:
>
> $$\mathcal{B}_{\text{box}} = \left\{ \prod_{\alpha \in A} U_\alpha : U_\alpha \in \mathcal{T}_\alpha \;\forall\, \alpha \right\}$$

Every coordinate is allowed to be a proper open subset simultaneously.

---

## 3. Box vs Product: The Key Difference

```
Comparison of basis elements (infinite product):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PRODUCT topology basis element:
  U₁ × U₂ × X₃ × X₄ × X₅ × X₆ × ...
  ╰──────────╯  ╰─────────────────────╯
  finitely many    ALL remaining coords
  constrained      are full Xₐ

BOX topology basis element:
  U₁ × U₂ × U₃ × U₄ × U₅ × U₆ × ...
  ╰──────────────────────────────────╯
  ALL coordinates can be proper open subsets
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

| Feature | Product Topology | Box Topology |
|---------|:---:|:---:|
| Basis elements constrain | Finitely many coords | All coords |
| For finite products | **Same** | **Same** |
| For infinite products | **Different** | **Different** |
| Contains the other | $\mathcal{T}_{\text{prod}} \subseteq \mathcal{T}_{\text{box}}$ | $\mathcal{T}_{\text{box}} \supseteq \mathcal{T}_{\text{prod}}$ |

> **✦ Key Insight:** For **finite** products, box = product. The difference only appears for **infinite** products. The box topology is strictly **finer** than the product topology.

---

## 4. Why the Box Topology Fails

### 4.1 Loss of Tychonoff's Theorem

> **Thm (Tychonoff).** The product of compact spaces is compact in the **product** topology.

This **fails** for the box topology:

**Counterexample:** $\prod_{n=1}^{\infty} [0,1]$ in the box topology is **not compact**.

The cover $\{U_n = \prod_{k=1}^{\infty} V_{n,k}\}$ where $V_{n,k} = [0,1]$ for $k \neq n$ and $V_{n,n} = [0, 1/2)$ or $(1/2, 1]$ appropriately chosen has no finite subcover.

### 4.2 Failure of Continuity of Natural Maps

The "diagonal" function $\Delta: \mathbb{R} \to \mathbb{R}^\omega$ defined by $\Delta(t) = (t, t, t, \ldots)$ is:

- **Continuous** in the product topology. ✓
- **NOT continuous** in the box topology. ✗

**Proof of failure:** Consider the box-open set $U = \prod_{n=1}^{\infty} (-1/n, 1/n)$.

$$\Delta^{-1}(U) = \{t : t \in (-1/n, 1/n) \;\forall\, n\} = \{0\}$$

The singleton $\{0\}$ is **not** open in $\mathbb{R}$. So $\Delta$ is not box-continuous. ∎

```
Failure of diagonal map continuity in box topology:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  U = (-1, 1) × (-1/2, 1/2) × (-1/3, 1/3) × ...
       │           │              │
       │           │              │  intervals shrink!
       ▼           ▼              ▼
  Δ⁻¹(U) = ℝ ∩ everything = {0}  ← not open!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 4.3 Convergence Pathology

The sequence $\mathbf{e}_n = (0, \ldots, 0, 1, 0, \ldots)$ (1 in position $n$):

- **Product topology:** $\mathbf{e}_n \to \mathbf{0}$ (each coordinate eventually becomes 0). ✓
- **Box topology:** $\mathbf{e}_n \not\to \mathbf{0}$ (take $U = \prod (-1/2, 1/2)$; $\mathbf{e}_n \notin U$ for all $n$). ✗

---

## 5. When Box = Product

The two topologies coincide when:
1. The index set $A$ is **finite**.
2. All but finitely many $X_\alpha$ have the **indiscrete** topology.

---

## 6. Properties Comparison

| Property | $\prod X_\alpha$ (Product) | $\prod X_\alpha$ (Box) |
|----------|:---:|:---:|
| Projections continuous | ✓ | ✓ |
| Universal property | ✓ | ✗ |
| Tychonoff theorem | ✓ | ✗ |
| Natural maps continuous | ✓ | ✗ (often) |
| Metrizable ($\omega$ factors) | ✓ | ✗ |
| Hausdorff | ✓ (if factors are) | ✓ (if factors are) |
| Connected | ✓ (if factors are) | ✗ (usually) |
| Normal | Depends | Very complex |

---

## 7. The Box Topology Has Its Uses

Despite its pathologies, the box topology is studied:

1. **Analyzing uniformity:** It detects differences that the product topology ignores.
2. **Counterexamples:** Many counterexamples in general topology use the box topology.
3. **Normality questions:** Whether $\mathbb{R}^\omega$ in the box topology is normal is a difficult problem (consistent with and independent of ZFC for some variants).

---

## 8. Summary Table

| Concept | Key Point |
|---------|-----------|
| Box topology basis | $\prod U_\alpha$ where every $U_\alpha$ open |
| Product topology basis | $\prod U_\alpha$ where all but finitely many $U_\alpha = X_\alpha$ |
| Finite products | Box = Product |
| Infinite products | Box strictly finer than Product |
| Tychonoff | Holds for product, fails for box |
| Diagonal map | Continuous for product, not for box |
| Universal property | Product has it; box does not |

---

## 9. Quick Revision Questions

1. Define the box topology and compare its basis with the product topology basis.

2. Show that the diagonal map $\Delta: \mathbb{R} \to \mathbb{R}^\omega$ is not continuous in the box topology.

3. Why does Tychonoff's theorem fail for the box topology? Sketch a proof idea.

4. When do the box and product topologies agree?

5. Is $\mathbb{R}^\omega$ connected in the box topology? Compare with the product topology.

6. Prove that the box topology on $\prod X_\alpha$ is always finer than the product topology.

---

[← Previous: Product Topology](01-product-topology.md) | [Back to README](../README.md) | [Next: Quotient Topology →](03-quotient-topology.md)
