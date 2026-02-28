# Chapter 4.4 — Local Connectedness

[← Previous: Components](03-components.md) | [Back to README](../README.md) | [Next: Compactness — Definition and Properties →](../05-Compactness/01-definition-and-properties.md)

---

## 1. Overview

A space can be connected "globally" but fail to behave well "locally." **Local connectedness** ensures every point has arbitrarily small connected neighborhoods. Crucially, in locally connected spaces, connected components are open and connectedness equals path-connectedness.

---

## 2. Definitions

> **Def 4.1 (Locally Connected).** $X$ is **locally connected** at $x$ if for every open $U$ containing $x$, there exists a connected open $V$ with $x \in V \subseteq U$. $X$ is **locally connected** if it is locally connected at every point.

> **Def 4.2 (Locally Path-Connected).** $X$ is **locally path-connected** at $x$ if for every open $U$ containing $x$, there exists a path-connected open $V$ with $x \in V \subseteq U$.

```
Local connectedness at x:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ╭──────── U (open) ──────╮
  │                         │
  │    ╭─── V ────╮         │
  │    │  (open,  │         │
  │    │ connected,│        │
  │    │   • x )  │         │
  │    ╰──────────╯         │
  ╰─────────────────────────╯

  For every U ∋ x, find connected open V ⊆ U
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 3. Key Relationships

```
Implication Diagram:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Path-connected ──→ Connected
       ↑                  ↑
  Locally path- ──→ Locally connected
  connected           

  Locally path-connected + connected ⟹ path-connected
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

| Relationship | True? |
|-------------|:---:|
| Locally connected ⟹ connected | ✗ ($\{0\} \cup \{1\}$ discrete) |
| Connected ⟹ locally connected | ✗ (topologist's sine curve) |
| Locally path-connected + connected ⟹ path-connected | ✓ |
| Locally path-connected ⟹ locally connected | ✓ |

---

## 4. Components Are Open in Locally Connected Spaces

> **Thm 4.3.** If $X$ is locally connected, then every connected component is **open** (hence also closed, i.e., clopen).

**Proof:** Let $C$ be a component and $x \in C$. By local connectedness, there is a connected open $V$ with $x \in V$. Since $V$ is connected and meets $C$, $V \subseteq C$ (maximality of $C$). So $C$ is a union of open sets, hence open. ∎

> **Cor 4.4.** A locally connected space has only finitely many components iff it is compact.

---

## 5. Characterization via Open Sets

> **Thm 4.5.** $X$ is locally connected if and only if for every open $U \subseteq X$, each connected component of $U$ is open in $X$.

This gives a clean characterization without reference to individual points.

---

## 6. Examples

| Space | Locally connected? | Locally path-connected? |
|-------|:---:|:---:|
| $\mathbb{R}^n$ | ✓ | ✓ |
| Open subsets of $\mathbb{R}^n$ | ✓ | ✓ |
| Topological manifolds | ✓ | ✓ |
| $\mathbb{Q}$ | ✗ | ✗ |
| Cantor set | ✗ | ✗ |
| Topologist's sine curve | ✗ | ✗ |
| Discrete spaces | ✓ | ✓ |
| The comb space | Connected but not locally connected at some points |

### The Comb Space

$$C = (\{0\} \times [0,1]) \cup ([0,1] \times \{0\}) \cup \bigcup_{n=1}^{\infty} (\{1/n\} \times [0,1])$$

```
Comb Space:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  1 ┤  │ │ │ │ │ │
    │  │ │ │ │ │ │  (teeth at x=1/n)
    │  │ │ │ │ │ │
    │  │ │ │ │ │ │
  0 ┤══╧═╧═╧═╧═╧═╧══════════
    0                        1
    │
    └── spine at x=0 and base at y=0
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

- Connected: Yes (it is path-connected).
- Locally connected at $(0, y)$ for $y > 0$: **No** — any small open set around $(0, 1/2)$ includes parts of nearby teeth that are not connected to each other in the local neighborhood.

---

## 7. Locally Connected and Quotients

> **Prop 4.6.** If $X$ is locally connected and $p: X \to Y$ is a quotient map, then $Y$ is locally connected.

This is because quotient maps preserve the "locally open connected neighborhoods" structure.

---

## 8. Summary Table

| Concept | Definition / Key Point |
|---------|----------------------|
| Locally connected | Every point has arbitrarily small connected open neighborhoods |
| Locally path-connected | Every point has arbitrarily small path-connected open neighborhoods |
| Loc. connected ⟹ components open | Always; components are clopen |
| Connected + loc. path-connected | ⟹ path-connected |
| $\mathbb{R}^n$ and manifolds | Locally path-connected ✓ |
| Topologist's sine curve | Connected but NOT locally connected |
| Comb space | Connected, not locally connected at some points |
| Quotient preserves | Local connectedness (under quotient maps) |

---

## 9. Quick Revision Questions

1. Define locally connected and locally path-connected. How are they related?

2. Prove that in a locally connected space, every connected component is open.

3. Give an example of a connected space that is not locally connected.

4. Show: connected + locally path-connected ⟹ path-connected.

5. Is the Cantor set locally connected? Justify.

6. Does a quotient of a locally connected space remain locally connected? Prove or give a reference.

---

[← Previous: Components](03-components.md) | [Back to README](../README.md) | [Next: Compactness — Definition and Properties →](../05-Compactness/01-definition-and-properties.md)
