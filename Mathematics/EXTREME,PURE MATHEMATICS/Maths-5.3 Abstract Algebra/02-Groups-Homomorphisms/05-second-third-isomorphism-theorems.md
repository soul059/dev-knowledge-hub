# Chapter 2.5 — Second and Third Isomorphism Theorems

[← Previous: First Isomorphism Theorem](04-first-isomorphism-theorem.md) | [Back to README](../README.md) | [Next: Cosets and Lagrange's Theorem →](../03-Groups-Structure/01-cosets-and-lagranges-theorem.md)

---

## Overview

The **Second** and **Third Isomorphism Theorems** are structural companions to the First. The Second (Diamond) Theorem relates the interplay of a subgroup and a normal subgroup, while the Third relates iterated quotients. Together they form the backbone of structural group theory.

---

## 1. Second Isomorphism Theorem (Diamond Theorem)

**Theorem 2.14.** Let $G$ be a group, $H \leq G$, and $N \trianglelefteq G$. Then:
1. $HN = \{hn : h \in H, n \in N\}$ is a subgroup of $G$.
2. $H \cap N \trianglelefteq H$.
3. $H / (H \cap N) \;\cong\; HN / N$.

```
  The Diamond (Lattice) Diagram:
  
              HN
             /  \
            /    \
           H      N         HN ≤ G
            \    /           H ∩ N ◁ H
             \  /
            H ∩ N            H/(H∩N) ≅ HN/N
```

### Proof Sketch

Define $\phi: H \to HN/N$ by $\phi(h) = hN$.
- **Well-defined:** $h \in H \subseteq HN$, so $hN \in HN/N$. ✓
- **Homomorphism:** $\phi(h_1h_2) = h_1h_2N = (h_1N)(h_2N) = \phi(h_1)\phi(h_2)$. ✓
- **Surjective:** Any $hnN \in HN/N$ equals $hN = \phi(h)$. ✓
- **Kernel:** $\phi(h) = N \iff h \in N \iff h \in H \cap N$. So $\ker\phi = H \cap N$.

By the FIT: $H/(H \cap N) \cong HN/N$. $\square$

### Example

Let $G = \mathbb{Z}_{12}$, $H = \langle 2 \rangle = \{0, 2, 4, 6, 8, 10\}$, $N = \langle 3 \rangle = \{0, 3, 6, 9\}$.

- $HN = \langle 2 \rangle + \langle 3 \rangle = \langle \gcd(2,3) \rangle = \langle 1 \rangle = \mathbb{Z}_{12}$
- $H \cap N = \langle \text{lcm}(2,3) \rangle = \langle 6 \rangle = \{0, 6\}$
- $H/(H \cap N) = \langle 2 \rangle / \langle 6 \rangle$: order $= 6/2 = 3$
- $HN/N = \mathbb{Z}_{12}/\langle 3 \rangle$: order $= 12/4 = 3$
- Both $\cong \mathbb{Z}_3$. ✓

```
  Example verification:
  
         Z₁₂           = HN
        /    \
    ⟨2⟩      ⟨3⟩        = H, N
      \      /
       ⟨6⟩              = H ∩ N
  
  H/(H∩N) = ⟨2⟩/⟨6⟩ ≅ Z₃
  HN/N = Z₁₂/⟨3⟩     ≅ Z₃    ✓
```

---

## 2. Third Isomorphism Theorem

**Theorem 2.15.** Let $G$ be a group with $N \trianglelefteq G$ and $M \trianglelefteq G$ such that $N \leq M$. Then:
1. $M/N \trianglelefteq G/N$.
2. $(G/N) / (M/N) \;\cong\; G/M$.

```
  Third Isomorphism Theorem:
  
       G
       │
       │  mod N
       ▼
      G/N
       │
       │  mod M/N
       ▼
   (G/N)/(M/N)  ≅  G/M
   
  "Quotienting out in two stages gives the same 
   result as quotienting out all at once."
   
  Analogy: (a/b)/c = a/(bc)  for fractions
           "cancelling N from numerator and denominator"
```

### Proof Sketch

Define $\phi: G/N \to G/M$ by $\phi(gN) = gM$.
- **Well-defined:** $g_1N = g_2N \implies g_1^{-1}g_2 \in N \subseteq M \implies g_1M = g_2M$. ✓
- **Surjective:** Clear. ✓
- **Kernel:** $\phi(gN) = M \iff g \in M \iff gN \in M/N$. So $\ker\phi = M/N$.

By the FIT: $(G/N)/(M/N) \cong G/M$. $\square$

### Example

$G = \mathbb{Z}$, $N = 6\mathbb{Z}$, $M = 2\mathbb{Z}$ (note $6\mathbb{Z} \subseteq 2\mathbb{Z}$).

- $G/N = \mathbb{Z}/6\mathbb{Z} \cong \mathbb{Z}_6$
- $M/N = 2\mathbb{Z}/6\mathbb{Z} \cong \{0, 2, 4\} \leq \mathbb{Z}_6$, which is $\cong \mathbb{Z}_3$
- $(G/N)/(M/N) = \mathbb{Z}_6/\{0, 2, 4\} \cong \mathbb{Z}_2$
- $G/M = \mathbb{Z}/2\mathbb{Z} \cong \mathbb{Z}_2$ ✓

---

## 3. All Three Theorems at a Glance

```
  ╔══════════════════════════════════════════════════════════╗
  ║  FIRST ISOMORPHISM THEOREM                              ║
  ║  φ: G → H homomorphism                                 ║
  ║  G/ker φ ≅ im φ                                        ║
  ╠══════════════════════════════════════════════════════════╣
  ║  SECOND ISOMORPHISM THEOREM (Diamond)                   ║
  ║  H ≤ G, N ◁ G                                          ║
  ║  H/(H ∩ N) ≅ HN/N                                     ║
  ╠══════════════════════════════════════════════════════════╣
  ║  THIRD ISOMORPHISM THEOREM                              ║
  ║  N ≤ M, both ◁ G                                       ║
  ║  (G/N)/(M/N) ≅ G/M                                    ║
  ╚══════════════════════════════════════════════════════════╝
```

---

## 4. The Zassenhaus Lemma (Butterfly Lemma)

A deeper result that generalises the Second Isomorphism Theorem:

**Theorem 2.16 (Zassenhaus).** Let $A' \trianglelefteq A \leq G$ and $B' \trianglelefteq B \leq G$. Then:

$$\frac{A'(A \cap B)}{A'(A \cap B')} \;\cong\; \frac{B'(A \cap B)}{B'(A' \cap B)} \;\cong\; \frac{A \cap B}{(A' \cap B)(A \cap B')}$$

```
  The Butterfly Diagram:
  
             A                    B
            / \                  / \
           /   \                /   \
       A'(A∩B) (A∩B)B'     A'(A∩B)  (A∩B)B'
         / \   / \            / ╲   / \
        /   \ /   \          /   ╲ /   \
    A'(A∩B') ═ (A'∩B)B'    central quotient
        \       /
         \     /
          A'∩B'
          
  ═ marks the isomorphic quotients
```

This lemma is used in proving the **Schreier Refinement Theorem** and the **Jordan-Hölder Theorem**.

---

## 5. Applying the Isomorphism Theorems — Worked Problems

### Problem 1: Identify $\mathbb{Z}_{24}/\langle 8 \rangle$

$\langle 8 \rangle = \{0, 8, 16\}$ in $\mathbb{Z}_{24}$, so $|\langle 8 \rangle| = 3$.

$|\mathbb{Z}_{24}/\langle 8 \rangle| = 24/3 = 8$.

Since $\mathbb{Z}_{24}/\langle 8 \rangle$ is a quotient of a cyclic group, it is cyclic: $\mathbb{Z}_{24}/\langle 8 \rangle \cong \mathbb{Z}_8$.

### Problem 2: Show $(A_4 \cap V_4) \trianglelefteq A_4$

$V_4 = \{e, (12)(34), (13)(24), (14)(23)\} \leq A_4$ and $V_4 \trianglelefteq A_4$.

By the Second Isomorphism Theorem with $H = A_4$, $N = V_4$ in $S_4$:
$A_4/(A_4 \cap V_4) \cong A_4 V_4 / V_4$

Since $V_4 \leq A_4$, we have $A_4 \cap V_4 = V_4$, so this simplifies to $A_4/V_4 \cong A_4/V_4$. The key point is $V_4 \trianglelefteq A_4$, giving $|A_4/V_4| = 12/4 = 3$, so $A_4/V_4 \cong \mathbb{Z}_3$.

### Problem 3: Iterated quotient via Third Isomorphism Theorem

Show $(\mathbb{Z}/12\mathbb{Z}) / (4\mathbb{Z}/12\mathbb{Z}) \cong \mathbb{Z}/4\mathbb{Z}$.

Set $G = \mathbb{Z}$, $N = 12\mathbb{Z}$, $M = 4\mathbb{Z}$. Then $N \subseteq M$ ✓.

By the Third Isomorphism Theorem:
$$(\mathbb{Z}/12\mathbb{Z})/(4\mathbb{Z}/12\mathbb{Z}) \cong \mathbb{Z}/4\mathbb{Z} \cong \mathbb{Z}_4$$✓

---

## 6. Connections and the Big Picture

```
  The isomorphism theorems connect:
  
  HOMOMORPHISMS ←──→ NORMAL SUBGROUPS ←──→ QUOTIENT GROUPS
  
  • Normal subgroups ARE kernels (Theorem 2.10)
  • Quotient groups ARE images (FIT)
  • All three isomorphism theorems follow from the FIT
  
  ═══════════════════════════════════════════════
  The FIT is the MASTER theorem; the 2nd, 3rd, 
  and Correspondence theorems are consequences.
  ═══════════════════════════════════════════════
```

---

## Summary Table

| Theorem | Hypotheses | Conclusion |
|---------|-----------|-----------|
| **First (FIT)** | $\phi: G \to H$ homomorphism | $G/\ker\phi \cong \text{im}\,\phi$ |
| **Second (Diamond)** | $H \leq G$, $N \trianglelefteq G$ | $H/(H \cap N) \cong HN/N$ |
| **Third** | $N \leq M$, both $\trianglelefteq G$ | $(G/N)/(M/N) \cong G/M$ |
| **Correspondence** | $N \trianglelefteq G$ | Subgroups of $G/N \leftrightarrow$ subgroups of $G$ containing $N$ |
| **Zassenhaus** | $A' \trianglelefteq A$, $B' \trianglelefteq B$ | $\frac{A'(A \cap B)}{A'(A \cap B')} \cong \frac{A \cap B}{(A' \cap B)(A \cap B')}$ |

---

## Quick Revision Questions

1. **State** the Second Isomorphism Theorem and draw the diamond diagram.

2. **Verify** the Second Isomorphism Theorem for $G = S_3$, $H = \langle(12)\rangle$, $N = A_3$.

3. **Use the Third Isomorphism Theorem** to show $(\mathbb{Z}/30\mathbb{Z})/(6\mathbb{Z}/30\mathbb{Z}) \cong \mathbb{Z}_6$.

4. **Explain** why the Third Isomorphism Theorem can be thought of as "cancelling" $N$ from $G/N$.

5. **Apply** the Correspondence Theorem to find all subgroups of $\mathbb{Z}/12\mathbb{Z}$ from the subgroups of $\mathbb{Z}$.

6. **State** all three isomorphism theorems and describe the proof strategy common to all of them.

---

[← Previous: First Isomorphism Theorem](04-first-isomorphism-theorem.md) | [Back to README](../README.md) | [Next: Cosets and Lagrange's Theorem →](../03-Groups-Structure/01-cosets-and-lagranges-theorem.md)
