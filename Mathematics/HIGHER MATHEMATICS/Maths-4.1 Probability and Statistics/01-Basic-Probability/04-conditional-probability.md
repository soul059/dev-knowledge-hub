# Chapter 1.4: Conditional Probability

[← Previous: Addition and Multiplication Rules](03-addition-and-multiplication-rules.md) | [Back to README](../README.md) | [Next: Independent Events →](05-independent-events.md)

---

## Overview

**Conditional probability** answers the question: "How does knowing that one event has occurred change the probability of another event?" It is one of the most important concepts in probability and forms the foundation for Bayes' theorem, Markov chains, and statistical inference.

---

## 1. Definition

The **conditional probability** of event $A$ given that event $B$ has occurred is:

$$\boxed{P(A \mid B) = \frac{P(A \cap B)}{P(B)}, \quad P(B) > 0}$$

### Interpretation

- The "given that $B$ occurred" effectively **reduces the sample space** from $S$ to $B$.
- We then ask: what fraction of $B$ also belongs to $A$?

```
  Before Conditioning (full S)       After Conditioning (given B)
  
  ┌─────────────── S ──────────┐     ┌──────── B (new S) ─────────┐
  │                             │     │                             │
  │   ┌────┐    ┌────┐         │     │         ┌────────────┐      │
  │   │ A  │    │ B  │         │     │         │  A ∩ B     │      │
  │   │  ┌─┼────┼─┐  │        │     │         │            │      │
  │   │  │ A∩B  │  │  │        │     │         │  P(A|B) =  │      │
  │   │  └─┼────┼─┘  │        │     │         │  P(A∩B)    │      │
  │   └────┘    └────┘         │     │         │  ───────   │      │
  │                             │     │         │   P(B)    │      │
  │                             │     │         └────────────┘      │
  └─────────────────────────────┘     └─────────────────────────────┘
  
  Full sample space S                 B becomes the new sample space
```

---

## 2. Intuitive Examples

### Example 1: Fair Die

Roll a fair die. $S = \{1,2,3,4,5,6\}$.

- $A$ = "number is ≤ 3" = $\{1, 2, 3\}$
- $B$ = "number is even" = $\{2, 4, 6\}$

Without conditioning: $P(A) = 3/6 = 1/2$

With conditioning on $B$:
$$P(A \mid B) = \frac{P(A \cap B)}{P(B)} = \frac{P(\{2\})}{P(\{2,4,6\})} = \frac{1/6}{3/6} = \frac{1}{3}$$

**Insight:** Knowing the number is even reduces the chance it is ≤ 3 (from 1/2 to 1/3).

### Example 2: Two Children Problem

A family has two children. At least one is a girl. What is the probability both are girls?

- $S = \{BB, BG, GB, GG\}$ (equally likely)
- $A$ = "both girls" = $\{GG\}$
- $B$ = "at least one girl" = $\{BG, GB, GG\}$

$$P(A \mid B) = \frac{P(A \cap B)}{P(B)} = \frac{P(\{GG\})}{P(\{BG, GB, GG\})} = \frac{1/4}{3/4} = \frac{1}{3}$$

---

## 3. Properties of Conditional Probability

Conditional probability $P(\cdot \mid B)$ is itself a valid probability measure (satisfies all Kolmogorov axioms):

| Property | Statement |
|----------|-----------|
| Non-negativity | $P(A \mid B) \geq 0$ |
| Normalization | $P(B \mid B) = 1$ |
| Additivity | If $A_1 \cap A_2 = \emptyset$, then $P(A_1 \cup A_2 \mid B) = P(A_1 \mid B) + P(A_2 \mid B)$ |
| Complement | $P(A^c \mid B) = 1 - P(A \mid B)$ |
| Sample space | $P(S \mid B) = 1$ |

---

## 4. Multiplication Rule (Revisited)

From the definition of conditional probability, we can derive the **multiplication rule**:

$$P(A \cap B) = P(B) \cdot P(A \mid B) = P(A) \cdot P(B \mid A)$$

### Chain Rule (Extended Multiplication)

For three events:
$$P(A \cap B \cap C) = P(A) \cdot P(B \mid A) \cdot P(C \mid A \cap B)$$

For $n$ events:
$$P\!\left(\bigcap_{i=1}^n A_i\right) = P(A_1) \cdot P(A_2 \mid A_1) \cdot P(A_3 \mid A_1 \cap A_2) \cdots P(A_n \mid A_1 \cap \cdots \cap A_{n-1})$$

---

## 5. Tree Diagrams

Tree diagrams provide a powerful visual tool for multi-stage probability problems.

### Example 3: Drawing Balls Without Replacement

A bag has 5 red (R) and 3 blue (B) balls. Two are drawn without replacement.

```
                             Start
                            /     \
                         5/8       3/8
                         /           \
                       R₁             B₁
                      / \            / \
                   4/7  3/7       5/7  2/7
                   /     \        /     \
                 R₂      B₂     R₂     B₂
                 
  Path Probabilities:
  ┌──────────────┬───────────────────────┬────────────────┐
  │   Path       │    Calculation        │  Probability   │
  ├──────────────┼───────────────────────┼────────────────┤
  │  R₁ → R₂    │  (5/8) × (4/7)       │   20/56        │
  │  R₁ → B₂    │  (5/8) × (3/7)       │   15/56        │
  │  B₁ → R₂    │  (3/8) × (5/7)       │   15/56        │
  │  B₁ → B₂    │  (3/8) × (2/7)       │    6/56        │
  ├──────────────┼───────────────────────┼────────────────┤
  │   Total      │                       │   56/56 = 1 ✓  │
  └──────────────┴───────────────────────┴────────────────┘
```

**Calculations:**
- $P(\text{both red}) = \frac{20}{56} = \frac{5}{14}$
- $P(\text{one of each}) = \frac{15}{56} + \frac{15}{56} = \frac{30}{56} = \frac{15}{28}$
- $P(\text{both blue}) = \frac{6}{56} = \frac{3}{28}$

---

## 6. Law of Total Probability

If $B_1, B_2, \ldots, B_n$ form a **partition** of $S$ (mutually exclusive and exhaustive), then for any event $A$:

$$\boxed{P(A) = \sum_{i=1}^n P(A \mid B_i) \cdot P(B_i)}$$

### Visual Interpretation

```
  ┌──────────────────────── S ────────────────────────────┐
  │     B₁      │      B₂      │      B₃     │    B₄     │
  │              │              │             │           │
  │   ┌─────────┼──────────────┼─────┐       │           │
  │   │ A∩B₁    │    A∩B₂      │A∩B₃ │       │           │
  │   │         │              │     │       │           │
  │   └─────────┼──────────────┼─────┘       │           │
  │              │              │             │           │
  └──────────────┴──────────────┴─────────────┴───────────┘
  
  P(A) = P(A∩B₁) + P(A∩B₂) + P(A∩B₃) + P(A∩B₄)
       = P(A|B₁)P(B₁) + P(A|B₂)P(B₂) + P(A|B₃)P(B₃) + P(A|B₄)P(B₄)
```

### Example 4: Manufacturing

Three machines produce bolts:
- Machine 1: Produces 30% of bolts, 4% defective
- Machine 2: Produces 45% of bolts, 5% defective
- Machine 3: Produces 25% of bolts, 3% defective

**Find:** $P(\text{randomly selected bolt is defective})$

Let $D$ = "defective", $M_i$ = "from machine $i$"

$$P(D) = P(D \mid M_1) P(M_1) + P(D \mid M_2) P(M_2) + P(D \mid M_3) P(M_3)$$

$$= (0.04)(0.30) + (0.05)(0.45) + (0.03)(0.25)$$

$$= 0.012 + 0.0225 + 0.0075 = 0.042$$

```
  Total Probability — Tree View
  
                    Start
                   /  |  \
               0.30  0.45  0.25
                /     |      \
              M₁     M₂      M₃
             / \    / \     / \
          0.04 0.96 0.05 0.95 0.03 0.97
           /     \   /     \   /     \
          D      D' D      D' D      D'
  
  P(D) = 0.30×0.04 + 0.45×0.05 + 0.25×0.03 = 0.042
```

---

## 7. Common Mistakes

| Mistake | Correct Statement |
|---------|-------------------|
| $P(A \mid B) = P(B \mid A)$ | Generally **FALSE**. $P(\text{wet}\mid\text{rain}) \neq P(\text{rain}\mid\text{wet})$ |
| $P(A \mid B) = P(A)$ always | Only true if $A$ and $B$ are **independent** |
| Forgetting to update after removal | Without replacement changes the denominator |
| $P(A \mid B) + P(A \mid B^c) = 1$ | **FALSE**. Instead: $P(A \mid B) + P(A^c \mid B) = 1$ |

### The Prosecutor's Fallacy

$P(\text{evidence} \mid \text{innocent})$ is NOT the same as $P(\text{innocent} \mid \text{evidence})$.

Example: $P(\text{DNA match} \mid \text{innocent}) = 1/1{,}000{,}000$ does NOT mean $P(\text{innocent} \mid \text{DNA match}) = 1/1{,}000{,}000$.

---

## 8. Real-World Applications

| Domain | Example |
|--------|---------|
| **Medicine** | $P(\text{disease} \mid \text{positive test})$ — diagnostic accuracy |
| **Spam Filtering** | $P(\text{spam} \mid \text{contains "free"})$ — Bayesian classifiers |
| **Weather** | $P(\text{rain tomorrow} \mid \text{cloudy today})$ |
| **Finance** | $P(\text{market crash} \mid \text{interest rate hike})$ |
| **Genetics** | $P(\text{carrier} \mid \text{parent is carrier})$ |
| **QC** | $P(\text{defective} \mid \text{from Machine A})$ |

---

## Summary Table

| Concept | Formula |
|---------|---------|
| Conditional Probability | $P(A \mid B) = \frac{P(A \cap B)}{P(B)}$ |
| Multiplication Rule | $P(A \cap B) = P(A) \cdot P(B \mid A)$ |
| Chain Rule (3 events) | $P(A \cap B \cap C) = P(A) \cdot P(B\mid A) \cdot P(C\mid A \cap B)$ |
| Law of Total Probability | $P(A) = \sum P(A\mid B_i) P(B_i)$ (partition $\{B_i\}$) |
| Complement given $B$ | $P(A^c \mid B) = 1 - P(A \mid B)$ |
| Key Warning | $P(A\mid B) \neq P(B \mid A)$ in general |

---

## Quick Revision Questions

1. A bag has 4 white and 6 black balls. Two balls are drawn without replacement. Find $P(\text{2nd is white} \mid \text{1st is black})$.

2. 1% of a population has a disease. A test has 99% sensitivity and 95% specificity. Find $P(\text{disease} \mid \text{positive test})$. (*This leads directly to Bayes' theorem — try it!*)

3. **Prove** that conditional probability satisfies Kolmogorov's axioms (i.e., $P(\cdot \mid B)$ is a valid probability measure).

4. Why is $P(A \mid B) \neq P(B \mid A)$ in general? Give a concrete numerical example showing this.

5. A box has 3 red and 7 blue balls. Three balls are drawn without replacement. Using the chain rule, find $P(\text{all three are blue})$.

6. Using the Law of Total Probability: A company buys hard drives from 3 suppliers (A: 50%, B: 30%, C: 20%). Failure rates are A: 2%, B: 4%, C: 1%. What is the overall probability of a hard drive failure?

---

[← Previous: Addition and Multiplication Rules](03-addition-and-multiplication-rules.md) | [Back to README](../README.md) | [Next: Independent Events →](05-independent-events.md)
