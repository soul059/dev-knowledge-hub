# Why Sequence Models

> **Unit 1, Chapter 1** — Understanding why sequential data requires specialized architectures and what makes temporal dependencies unique.

---

## 📋 Overview

Not all data is created equal. While images have spatial structure and tabular data has independent features, much of the world's most valuable data is **sequential** — it comes in ordered sequences where the position and context of each element carry meaning. Sequence models are designed to capture these **temporal dependencies**.

---

## 🌍 Sequential Data Is Everywhere

### Types of Sequential Data

```
┌─────────────────────────────────────────────────────────────────┐
│                    SEQUENTIAL DATA TYPES                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📝 Text/Language                                               │
│     "The cat sat on the ___"                                    │
│      t₁   t₂  t₃  t₄  t₅                                      │
│     Each word depends on previous words                         │
│                                                                 │
│  🎵 Audio/Speech                                                │
│     ┌─┐┌──┐┌─┐┌───┐┌──┐                                       │
│     │ ││  ││ ││   ││  │  Waveform samples over time            │
│     └─┘└──┘└─┘└───┘└──┘                                       │
│                                                                 │
│  📈 Time Series                                                 │
│     Stock prices: $150 → $152 → $148 → $155 → ?                │
│     Each value depends on historical pattern                    │
│                                                                 │
│  🎬 Video                                                       │
│     Frame₁ → Frame₂ → Frame₃ → Frame₄                         │
│     Motion and action unfold over time                          │
│                                                                 │
│  🧬 Biological Sequences                                        │
│     DNA: A-T-G-C-C-T-A-G                                       │
│     Protein: Met-Ala-Gly-Ser                                    │
│                                                                 │
│  🎮 Actions/Events                                              │
│     User clicks: Search → Product → Cart → Buy                 │
│     Game moves: e4 → e5 → Nf3 → Nc6                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔗 What Are Temporal Dependencies?

A **temporal dependency** means that the meaning or value of an element depends on what came before (and sometimes after) it.

### Example: Language

```
Sentence: "The clouds are dark. I think it will ___"

Without context:  ??? (could be anything)
With context:     "rain" (high probability)

The word at position t depends on words at positions t-1, t-2, ..., t-k
```

### Example: Stock Prices

```
Day:    1     2     3     4     5     6
Price: $100  $102  $105  $103  $107  $???

The price at day 6 depends on the PATTERN of previous days,
not just day 5 alone. This is a temporal dependency.
```

### Short-Term vs Long-Term Dependencies

```
SHORT-TERM: "The cat sat on the mat"
             ───────────────▶ ───
             "mat" depends on nearby "sat on"

LONG-TERM:  "I grew up in France. I spent many years there.
             ... (20 sentences later) ...
             I speak fluent ___"
             ─────────────────────────────────▶
             "French" depends on "France" from far back
```

---

## 📊 Examples of Sequence Tasks

| Task | Input | Output | Type |
|------|-------|--------|------|
| Sentiment Analysis | Text sequence | Positive/Negative | Many-to-One |
| Machine Translation | English sentence | French sentence | Many-to-Many |
| Speech Recognition | Audio waveform | Text transcript | Many-to-Many |
| Music Generation | Seed notes | New melody | One-to-Many |
| Stock Prediction | Price history | Next price | Many-to-One |
| Video Captioning | Frame sequence | Text description | Many-to-One |
| Named Entity Recognition | Word sequence | Label sequence | Many-to-Many |
| Next Word Prediction | Word context | Next word | Many-to-One |

---

## 🧮 Mathematical Formulation

### Sequence Notation

A sequence of length T:

```
x = (x₁, x₂, x₃, ..., x_T)

where x_t ∈ ℝᵈ is the input at time step t
      d is the dimensionality of each input
      T is the sequence length (can vary!)
```

### The Prediction Problem

Given a sequence, we want to model:

```
P(y | x₁, x₂, ..., x_T)

or for generation:

P(x_{t+1} | x₁, x₂, ..., x_t)

The key challenge: How do we encode the variable-length
history (x₁, ..., x_t) into a fixed representation?
```

### Joint Probability (Autoregressive Decomposition)

```
P(x₁, x₂, ..., x_T) = P(x₁) · P(x₂|x₁) · P(x₃|x₁,x₂) · ... · P(x_T|x₁,...,x_{T-1})

                       T
                     = ∏ P(x_t | x₁, ..., x_{t-1})
                      t=1
```

---

## 💻 Python Example: Why Order Matters

```python
import numpy as np

# Same words, different order → different meaning
sentence1 = ["dog", "bites", "man"]    # Not newsworthy
sentence2 = ["man", "bites", "dog"]    # Very newsworthy!

# A bag-of-words model sees these as IDENTICAL
def bag_of_words(sentence, vocab):
    """Order-agnostic representation"""
    bow = np.zeros(len(vocab))
    for word in sentence:
        if word in vocab:
            bow[vocab.index(word)] = 1
    return bow

vocab = ["dog", "bites", "man"]
bow1 = bag_of_words(sentence1, vocab)
bow2 = bag_of_words(sentence2, vocab)

print(f"BoW sentence 1: {bow1}")  # [1, 1, 1]
print(f"BoW sentence 2: {bow2}")  # [1, 1, 1]
print(f"Identical? {np.array_equal(bow1, bow2)}")  # True!
# BoW loses all ordering information — we need sequence models!

# Time series example: order is everything
np.random.seed(42)
prices = [100]
for _ in range(10):
    prices.append(prices[-1] + np.random.randn() * 2)

print(f"\nOriginal prices: {[f'{p:.1f}' for p in prices]}")
print(f"Shuffled prices: {[f'{p:.1f}' for p in np.random.permutation(prices)]}")
print("Shuffled prices lose all trend information!")
```

---

## 🔑 Why We Can't Use Standard Neural Networks

```
Standard Feedforward Network:
┌──────────────┐
│ Fixed input   │   • Input size must be fixed
│ x ∈ ℝⁿ       │   • No notion of "time" or "order"
│              │   • Each input processed independently
│   W₁, W₂    │   • Cannot handle variable-length sequences
│              │   • No memory of previous inputs
│ Fixed output  │
│ y ∈ ℝᵐ       │
└──────────────┘

What we NEED:
┌──────────────┐
│ Variable      │   • Accept any sequence length
│ input length  │   • Understand temporal order
│              │   • Remember relevant past information
│ Memory of    │   • Share knowledge across time steps
│ past inputs  │   • Handle dependencies at any distance
│              │
│ Flexible      │
│ output        │
└──────────────┘
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Sequential Data | Data where order and position carry meaning |
| Temporal Dependency | Current value depends on past values |
| Short-term Dependency | Dependencies between nearby elements |
| Long-term Dependency | Dependencies spanning many time steps |
| Autoregressive Model | Predicts next element from previous elements |
| Variable Length | Sequences can have different lengths |
| Key Challenge | Encoding variable-length history into useful representation |

---

## ❓ Revision Questions

1. **Name four types of sequential data and explain why order matters for each.**

2. **What is the difference between short-term and long-term temporal dependencies? Give an example of each in natural language.**

3. **Write the autoregressive decomposition of P(x₁, x₂, x₃, x₄). How many conditional distributions need to be modeled?**

4. **Why does a bag-of-words representation fail for the sentences "dog bites man" and "man bites dog"?**

5. **List three properties that a sequence model must have that a standard feedforward network lacks.**

6. **A weather station records temperature every hour. Why is this sequential data, and what kind of dependencies might exist?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬆️ Unit Overview | [README](../README.md) |
| ➡️ Next | [Types of Sequence Problems](02-types-of-sequence-problems.md) |
