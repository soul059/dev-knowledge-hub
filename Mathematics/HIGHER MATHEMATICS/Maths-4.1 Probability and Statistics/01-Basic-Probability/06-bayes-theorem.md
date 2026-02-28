# Chapter 1.6: Bayes' Theorem

[← Previous: Independent Events](05-independent-events.md) | [Back to README](../README.md) | [Next: Fundamental Counting Principle →](../02-Permutations-and-Combinations/01-fundamental-counting-principle.md)

---

## Overview

**Bayes' theorem** is one of the most powerful and widely applied results in probability. It allows us to "reverse" conditional probabilities — given $P(B \mid A)$, we can find $P(A \mid B)$. It answers the fundamental question: *"How should we update our beliefs in light of new evidence?"*

---

## 1. Statement of Bayes' Theorem

### Simple Form (Two Events)

If $P(A) > 0$ and $P(B) > 0$:

$$\boxed{P(A \mid B) = \frac{P(B \mid A) \cdot P(A)}{P(B)}}$$

### General Form (Partition)

If $A_1, A_2, \ldots, A_n$ form a **partition** of $S$ (mutually exclusive and exhaustive), then:

$$\boxed{P(A_i \mid B) = \frac{P(B \mid A_i) \cdot P(A_i)}{\sum_{j=1}^{n} P(B \mid A_j) \cdot P(A_j)}}$$

where the denominator is the **Law of Total Probability**.

---

## 2. Terminology

| Term | Symbol | Meaning |
|------|--------|---------|
| **Prior probability** | $P(A_i)$ | Our belief about $A_i$ **before** observing evidence $B$ |
| **Likelihood** | $P(B \mid A_i)$ | How likely the evidence $B$ is if $A_i$ is true |
| **Posterior probability** | $P(A_i \mid B)$ | Updated belief about $A_i$ **after** observing $B$ |
| **Evidence (Marginal)** | $P(B)$ | Total probability of observing $B$ |

### The Bayesian Update Process

```
  ┌──────────────┐     Observe     ┌──────────────┐
  │    PRIOR      │    Evidence     │  POSTERIOR    │
  │   P(Aᵢ)      │ ──── B ────►   │  P(Aᵢ | B)   │
  │               │                │               │
  │ (Before data) │   Bayes'       │ (After data)  │
  │               │   Theorem      │               │
  └──────────────┘                 └──────────────┘
  
                    P(B|Aᵢ) · P(Aᵢ)
  P(Aᵢ | B) = ─────────────────────
                       P(B)
  
  Posterior ∝ Likelihood × Prior
```

---

## 3. Derivation

Starting from the definition of conditional probability:

$$P(A \mid B) = \frac{P(A \cap B)}{P(B)}$$

Using the multiplication rule: $P(A \cap B) = P(B \mid A) \cdot P(A)$

$$P(A \mid B) = \frac{P(B \mid A) \cdot P(A)}{P(B)}$$

Expanding $P(B)$ using the Law of Total Probability:

$$P(B) = P(B \mid A)P(A) + P(B \mid A^c)P(A^c)$$

Therefore:

$$P(A \mid B) = \frac{P(B \mid A) \cdot P(A)}{P(B \mid A) \cdot P(A) + P(B \mid A^c) \cdot P(A^c)}$$

---

## 4. Classic Example: Medical Diagnosis

### Problem

A disease affects 1% of the population. A diagnostic test has:
- **Sensitivity** (true positive rate): $P(\text{Positive} \mid \text{Disease}) = 0.99$
- **Specificity** (true negative rate): $P(\text{Negative} \mid \text{No Disease}) = 0.95$

A person tests positive. What is the probability they actually have the disease?

### Solution

**Define events:**
- $D$ = "has disease", $D^c$ = "no disease"
- $+$ = "tests positive", $-$ = "tests negative"

**Given:**
- $P(D) = 0.01$ (prior)
- $P(+ \mid D) = 0.99$ (sensitivity)
- $P(- \mid D^c) = 0.95 \implies P(+ \mid D^c) = 0.05$ (false positive rate)

**Apply Bayes' theorem:**

$$P(D \mid +) = \frac{P(+ \mid D) \cdot P(D)}{P(+ \mid D) \cdot P(D) + P(+ \mid D^c) \cdot P(D^c)}$$

$$= \frac{(0.99)(0.01)}{(0.99)(0.01) + (0.05)(0.99)}$$

$$= \frac{0.0099}{0.0099 + 0.0495} = \frac{0.0099}{0.0594} \approx 0.1667$$

### Result: Only about 16.7% chance of actually having the disease!

```
  Visualization: Why Positive ≠ Disease
  
  Population = 10,000 people
  
  ┌──────────────────────────────────────────────────────────────┐
  │  Disease (1%)          No Disease (99%)                      │
  │  100 people            9,900 people                          │
  │                                                              │
  │  Test +: 99            Test +: 495       ← FALSE POSITIVES   │
  │  Test -: 1             Test -: 9,405                         │
  │                                                              │
  │  Total Positives = 99 + 495 = 594                            │
  │                                                              │
  │  P(Disease | +) = 99/594 = 0.167                             │
  │                                                              │
  │  The 495 false positives SWAMP the 99 true positives!        │
  └──────────────────────────────────────────────────────────────┘
```

### Natural Frequency Table

| | Disease (D) | No Disease (D^c) | Total |
|---|:-----------:|:-----------------:|:-----:|
| Test + | 99 | 495 | **594** |
| Test − | 1 | 9,405 | 9,406 |
| **Total** | **100** | **9,900** | **10,000** |

---

## 5. Example 2: Manufacturing (Machines)

Three machines produce items:

| Machine | Production Share | Defect Rate |
|---------|:----------------:|:-----------:|
| $M_1$ | 30% | 4% |
| $M_2$ | 45% | 5% |
| $M_3$ | 25% | 3% |

An item is found defective. Which machine most likely produced it?

**Using Bayes' theorem:**

$$P(M_i \mid D) = \frac{P(D \mid M_i) \cdot P(M_i)}{P(D)}$$

From the Law of Total Probability:
$$P(D) = (0.04)(0.30) + (0.05)(0.45) + (0.03)(0.25) = 0.012 + 0.0225 + 0.0075 = 0.042$$

| Machine | $P(D \mid M_i) \cdot P(M_i)$ | $P(M_i \mid D)$ |
|---------|:-----------------------------:|:----------------:|
| $M_1$ | $(0.04)(0.30) = 0.012$ | $0.012/0.042 = 2/7 \approx 0.286$ |
| $M_2$ | $(0.05)(0.45) = 0.0225$ | $0.0225/0.042 = 15/28 \approx 0.536$ |
| $M_3$ | $(0.03)(0.25) = 0.0075$ | $0.0075/0.042 = 5/28 \approx 0.179$ |
| **Total** | **0.042** | **1.000** ✓ |

**Answer:** Machine 2 most likely produced the defective item (53.6%).

```
  Bayes' Update — Prior to Posterior
  
  Prior P(Mᵢ):          Posterior P(Mᵢ|D):
  
  M₁ ████████████ 30%    M₁ ██████████ 28.6%
  M₂ █████████████████ 45%    M₂ █████████████████████ 53.6%
  M₃ █████████ 25%    M₃ ██████ 17.9%
  
  Machine 2's share INCREASED because its defect rate is highest.
```

---

## 6. Example 3: The Monty Hall Problem

You're on a game show with 3 doors. Behind one door is a car; behind the others, goats. You pick Door 1. The host, who knows what's behind the doors, opens Door 3 (which has a goat). Should you switch to Door 2?

**Define:**
- $C_i$ = "car is behind door $i$" → $P(C_1) = P(C_2) = P(C_3) = 1/3$
- $H_3$ = "host opens door 3"

**Using Bayes' theorem:**

$P(H_3 \mid C_1) = 1/2$ (host can open either door 2 or 3)  
$P(H_3 \mid C_2) = 1$ (host must open door 3)  
$P(H_3 \mid C_3) = 0$ (host won't reveal the car)

$$P(C_1 \mid H_3) = \frac{P(H_3 \mid C_1) \cdot P(C_1)}{P(H_3)} = \frac{(1/2)(1/3)}{1/2} = \frac{1}{3}$$

$$P(C_2 \mid H_3) = \frac{P(H_3 \mid C_2) \cdot P(C_2)}{P(H_3)} = \frac{(1)(1/3)}{1/2} = \frac{2}{3}$$

where $P(H_3) = (1/2)(1/3) + (1)(1/3) + (0)(1/3) = 1/2$

**Conclusion:** You should **always switch** — switching gives $2/3$ probability vs. staying at $1/3$.

```
  Monty Hall — Decision Tree
  
  You pick Door 1
  ┌──────────┬──────────┬──────────┐
  │  Car @1  │  Car @2  │  Car @3  │
  │  (1/3)   │  (1/3)   │  (1/3)   │
  │          │          │          │
  │ Host     │ Host     │ Host     │
  │ opens    │ MUST     │ MUST     │
  │ 2 or 3   │ open 3   │ open 2   │
  │ (50/50)  │ (100%)   │ (100%)   │
  │          │          │          │
  │ Switch:  │ Switch:  │ Switch:  │
  │  LOSE    │  WIN     │  WIN     │
  └──────────┴──────────┴──────────┘
  
  Switch wins: 2/3    Stay wins: 1/3
```

---

## 7. The Odds Form of Bayes' Theorem

An elegant alternative formulation uses **odds**:

$$\frac{P(A \mid B)}{P(A^c \mid B)} = \frac{P(A)}{P(A^c)} \times \frac{P(B \mid A)}{P(B \mid A^c)}$$

$$\text{Posterior Odds} = \text{Prior Odds} \times \text{Likelihood Ratio}$$

### Example: Medical Test (Odds Form)

- Prior odds of disease = $P(D)/P(D^c) = 0.01/0.99 \approx 1:99$
- Likelihood ratio = $P(+|D)/P(+|D^c) = 0.99/0.05 = 19.8$
- Posterior odds = $(1/99) \times 19.8 = 19.8/99 = 0.2 \approx 1:5$

So $P(D|+) = 1/(1+5) \approx 0.167$ ✓ (matches our earlier answer)

---

## 8. Sequential Bayesian Updating

Bayes' theorem can be applied **repeatedly** as new evidence arrives:

$$P(A \mid B_1, B_2) = \frac{P(B_2 \mid A, B_1) \cdot P(A \mid B_1)}{P(B_2 \mid B_1)}$$

The **posterior** from the first update becomes the **prior** for the second update.

```
  Sequential Updating
  
  Prior    ──► Evidence B₁ ──► Posterior₁ ──► Evidence B₂ ──► Posterior₂
  P(A)         Bayes'          P(A|B₁)        Bayes'          P(A|B₁,B₂)
               Theorem                        Theorem
  
  Each new piece of evidence refines our belief.
```

---

## 9. Real-World Applications

| Domain | Application |
|--------|------------|
| **Medicine** | Diagnostic test interpretation, disease screening |
| **Spam Filtering** | Naive Bayes classifiers in email filtering |
| **Law** | Evaluating DNA evidence, forensic analysis |
| **AI / Machine Learning** | Bayesian networks, probabilistic reasoning |
| **Search & Rescue** | Updating location probability based on search results |
| **Finance** | Updating risk assessment with new market data |
| **Quality Control** | Identifying which machine produced a defect |
| **Epidemiology** | Estimating disease prevalence from test results |

---

## Summary Table

| Concept | Formula / Key Idea |
|---------|-------------------|
| Bayes' Theorem (simple) | $P(A \mid B) = \frac{P(B \mid A) \cdot P(A)}{P(B)}$ |
| Bayes' Theorem (partition) | $P(A_i \mid B) = \frac{P(B \mid A_i) P(A_i)}{\sum_j P(B \mid A_j) P(A_j)}$ |
| Prior | $P(A)$ — belief before evidence |
| Likelihood | $P(B \mid A)$ — probability of evidence given hypothesis |
| Posterior | $P(A \mid B)$ — updated belief after evidence |
| Evidence (marginal) | $P(B) = \sum P(B \mid A_j) P(A_j)$ |
| Odds Form | Posterior Odds = Prior Odds × Likelihood Ratio |
| Sequential Update | Posterior becomes prior for next evidence |
| Base Rate Fallacy | Ignoring $P(A)$ leads to wrong conclusions |

---

## Quick Revision Questions

1. A factory has two plants. Plant 1 produces 60% of items with 3% defect rate; Plant 2 produces 40% with 5% defect rate. A defective item is found. What is the probability it came from Plant 1?

2. **Explain** the base rate fallacy using the medical diagnosis example. Why is $P(\text{disease} \mid \text{positive})$ so low even when the test is 99% accurate?

3. A student knows 70% of the material. On a multiple-choice question (4 options), if the student knows the answer, they answer correctly. If not, they guess randomly. Given that the student answered correctly, what is the probability they actually knew the answer?

4. **Derive** Bayes' theorem from the definition of conditional probability.

5. In the Monty Hall problem, what happens if there are 100 doors (1 car, 99 goats), you pick one, and the host opens 98 goat doors? Should you switch? What is $P(\text{win})$ if you switch?

6. A rare event has prior probability 0.001. Two independent tests both give positive results. Test 1 has sensitivity 0.95 and specificity 0.90. Test 2 has sensitivity 0.90 and specificity 0.95. Using sequential Bayesian updating, find the posterior probability after both tests.

---

[← Previous: Independent Events](05-independent-events.md) | [Back to README](../README.md) | [Next: Fundamental Counting Principle →](../02-Permutations-and-Combinations/01-fundamental-counting-principle.md)
