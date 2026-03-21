# Chapter 1: N-gram Language Models

> **Unit 4.1 — NLP / 07 – Language Models** | [← Previous: Neural Sequence Labeling](../06-Sequence-Labeling/06-neural-sequence-labeling.md) | [Next → Perplexity](./02-perplexity.md)

---

## 1. Overview

A **language model** assigns a probability to a sequence of words. Given a sentence W = w₁ w₂ ... wₙ, a language model computes P(W) = P(w₁, w₂, ..., wₙ), answering the question: *"How likely is this sequence of words in natural language?"* N-gram models are the simplest and historically most important approach — they estimate these probabilities by counting word sequences in a corpus and applying the **Markov assumption** to make computation tractable. Despite being largely superseded by neural models, n-grams remain fundamental to understanding language modeling, smoothing, and evaluation metrics like perplexity.

---

## 2. What Is a Language Model?

A language model defines a **probability distribution over word sequences**. It serves two core functions:

1. **Scoring:** Assign a probability to a complete sentence  
   - P("the cat sat on the mat") > P("mat the on sat cat the")
2. **Prediction:** Predict the next word given a history  
   - P(w | "the cat sat on the") → most likely w = "mat"

### 2.1 Applications

| Application            | How the LM Is Used                                    |
|------------------------|-------------------------------------------------------|
| Speech Recognition     | Choose word sequence with highest P(W) given acoustics|
| Machine Translation    | Rank fluent translations by P(W)                      |
| Spelling Correction    | Pick correction that maximizes P(sentence)            |
| Text Generation        | Sample next word from P(wₙ | w₁...wₙ₋₁)             |
| Auto-complete          | Suggest most probable next words                      |

### 2.2 The Chain Rule of Probability

Any joint probability can be decomposed using the **chain rule**:

```
P(w₁, w₂, ..., wₙ) = P(w₁) · P(w₂|w₁) · P(w₃|w₁,w₂) · ... · P(wₙ|w₁,...,wₙ₋₁)

                     n
                  = ∏  P(wᵢ | w₁, w₂, ..., wᵢ₋₁)
                    i=1
```

**Example:** P("the cat sat") = P(the) · P(cat|the) · P(sat|the, cat)

The problem: computing P(wₙ | w₁, ..., wₙ₋₁) exactly requires seeing every possible history — which is infeasible for long sentences. This is where the **n-gram approximation** comes in.

---

## 3. The N-gram Approximation (Markov Assumption)

Instead of conditioning on the **entire** history, we approximate by conditioning on only the last **(n−1)** words:

```
┌─────────────────────────────────────────────────────────────────────┐
│  Full:     P(wₙ | w₁, w₂, ..., wₙ₋₁)                             │
│                                                                     │
│  N-gram:   P(wₙ | wₙ₋₍ₙ₋₁₎, ..., wₙ₋₁)   ← keep last (n-1)     │
│                                                 words only          │
│                                                                     │
│  Bigram:   P(wₙ | wₙ₋₁)                    ← 1st order Markov     │
│  Trigram:  P(wₙ | wₙ₋₂, wₙ₋₁)              ← 2nd order Markov    │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.1 Unigram Model (n = 1)

Each word is **independent** — no context at all:

```
P(w₁, w₂, ..., wₙ) = ∏ P(wᵢ)

P(wᵢ) = Count(wᵢ) / N       where N = total word count in corpus
```

### 3.2 Bigram Model (n = 2)

Each word depends on only the **previous** word:

```
P(w₁, w₂, ..., wₙ) = P(w₁) · ∏ P(wᵢ | wᵢ₋₁)

P(wᵢ | wᵢ₋₁) = Count(wᵢ₋₁, wᵢ) / Count(wᵢ₋₁)
```

### 3.3 Trigram Model (n = 3)

Each word depends on the **two preceding** words:

```
P(w₁, w₂, ..., wₙ) = P(w₁) · P(w₂|w₁) · ∏ P(wᵢ | wᵢ₋₂, wᵢ₋₁)

P(wᵢ | wᵢ₋₂, wᵢ₋₁) = Count(wᵢ₋₂, wᵢ₋₁, wᵢ) / Count(wᵢ₋₂, wᵢ₋₁)
```

---

## 4. N-gram Window — Visual Walkthrough

Here is how a **trigram** (n = 3) window slides over a sentence to extract context:

```
Sentence:  <s>   the   cat   sat   on   the   mat   </s>

Step 1:   [<s>   the   cat]  → P(cat | <s>, the)
               ──────────
Step 2:         [the   cat   sat]  → P(sat | the, cat)
                ──────────
Step 3:               [cat   sat   on]  → P(on | cat, sat)
                       ──────────
Step 4:                     [sat   on   the]  → P(the | sat, on)
                             ──────────
Step 5:                           [on   the   mat]  → P(mat | on, the)
                                   ──────────
Step 6:                                 [the   mat   </s>]  → P(</s> | the, mat)
                                         ──────────

Window size = n = 3 words
Context     = n - 1 = 2 preceding words
Slides      = one word at a time →
```

Note: `<s>` and `</s>` are special **start** and **end-of-sentence** tokens added during preprocessing.

---

## 5. Maximum Likelihood Estimation (MLE)

MLE estimates n-gram probabilities by **counting** occurrences in a training corpus:

```
                    Count(wₙ₋₁, wₙ)
P_MLE(wₙ|wₙ₋₁) = ──────────────────     (bigram MLE)
                      Count(wₙ₋₁)

                    Count(wₙ₋₂, wₙ₋₁, wₙ)
P_MLE(wₙ|wₙ₋₂,wₙ₋₁) = ──────────────────────     (trigram MLE)
                          Count(wₙ₋₂, wₙ₋₁)
```

### 5.1 Worked Example

**Corpus:** "the cat sat on the mat . the cat ate ."

| Bigram         | Count | P(w₂\|w₁)         |
|----------------|-------|--------------------|
| (the, cat)     | 2     | 2/3 ≈ 0.667       |
| (the, mat)     | 1     | 1/3 ≈ 0.333       |
| (cat, sat)     | 1     | 1/2 = 0.500       |
| (cat, ate)     | 1     | 1/2 = 0.500       |
| (sat, on)      | 1     | 1/1 = 1.000       |
| (on, the)      | 1     | 1/1 = 1.000       |

Count("the") = 3, so P(cat|the) = Count(the, cat)/Count(the) = 2/3

### 5.2 The Zero-Probability Problem

MLE assigns **P = 0** to any n-gram not seen in training:

```
P_MLE(unicorn | the) = Count(the, unicorn) / Count(the) = 0/3 = 0

⟹ Entire sentence probability becomes 0 if it contains
   even ONE unseen n-gram!
```

This is catastrophic — and motivates **smoothing**.

---

## 6. Smoothing Techniques

Smoothing redistributes probability mass from seen n-grams to unseen ones, ensuring no n-gram has zero probability.

### 6.1 Laplace (Add-1) Smoothing

Add 1 to every count:

```
                    Count(wₙ₋₁, wₙ) + 1
P_Lap(wₙ|wₙ₋₁) = ────────────────────────
                    Count(wₙ₋₁) + V

Where V = vocabulary size (number of unique word types)
```

**Intuition:** Pretend we saw every possible bigram at least once.

| Pros                          | Cons                                        |
|-------------------------------|---------------------------------------------|
| Simple to implement           | Shifts too much mass to unseen events       |
| Guarantees no zero probs      | Drastically changes counts for rare n-grams |
| Works well for text classif.  | Poor for language modeling in practice       |

### 6.2 Add-k Smoothing

Generalization of Laplace — add a fractional count k (0 < k < 1):

```
                    Count(wₙ₋₁, wₙ) + k
P_k(wₙ|wₙ₋₁) = ────────────────────────
                  Count(wₙ₋₁) + k·V

Typical k values: 0.01, 0.1, 0.5
k is tuned on a held-out (validation) set.
```

### 6.3 Kneser-Ney Smoothing

The **gold standard** for n-gram smoothing. Key insight: the probability of a word should depend not just on its count, but on the **number of different contexts** it appears in (its **versatility**).

```
                          max(Count(wₙ₋₁, wₙ) − d, 0)        λ(wₙ₋₁)
P_KN(wₙ|wₙ₋₁) = ───────────────────────────────── + ─────────── · P_cont(wₙ)
                            Count(wₙ₋₁)                Count(wₙ₋₁)

Where:
  d = fixed discount (typically 0.75)
  λ(wₙ₋₁) = normalizing constant = (d / Count(wₙ₋₁)) · |{w : Count(wₙ₋₁, w) > 0}|

              |{wᵢ₋₁ : Count(wᵢ₋₁, wₙ) > 0}|
P_cont(wₙ) = ──────────────────────────────────
              |{(wⱼ, wₖ) : Count(wⱼ, wₖ) > 0}|
```

**Intuition with an example:**

```
"San Francisco" appears 1000 times in corpus.
"Francisco" almost always follows "San."

MLE says:  P(Francisco) is high      (high raw count)
KN  says:  P(Francisco) is low       (appears in only 1 context: "San ___")

Word like "the" appears in MANY different contexts → high P_cont
Word like "Francisco" appears in FEW contexts       → low  P_cont
```

### 6.4 Smoothing Comparison

| Method       | Formula Adjustment  | Strength                    | Weakness                     |
|--------------|--------------------|-----------------------------|------------------------------|
| Laplace      | +1 to all counts   | Dead simple                 | Distorts distribution heavily|
| Add-k        | +k to all counts   | Tunable, less distortion    | k requires validation set    |
| Kneser-Ney   | Discount + backoff  | Best perplexity scores      | Complex to implement         |

---

## 7. Computing Sentence Probability

To score a full sentence, multiply all the conditional n-gram probabilities. In practice, we use **log probabilities** to avoid floating-point underflow:

```
                  n
log P(W) = Σ  log P(wᵢ | wᵢ₋₁)        (for bigram model)
                 i=1

Example: P("the cat sat") using bigrams

  = P(the|<s>) · P(cat|the) · P(sat|cat) · P(</s>|sat)

log P = log P(the|<s>) + log P(cat|the) + log P(sat|cat) + log P(</s>|sat)
```

Using log probabilities turns products into sums, which is **numerically stable** and computationally efficient.

---

## 8. Python Implementation — Bigram Language Model

```python
from collections import Counter, defaultdict
import math

class BigramLanguageModel:
    """A bigram language model with add-k smoothing."""

    def __init__(self, k=0.01):
        self.k = k
        self.bigram_counts = Counter()
        self.unigram_counts = Counter()
        self.vocab = set()

    def train(self, corpus):
        """Train on a list of sentences (each sentence is a list of words)."""
        for sentence in corpus:
            tokens = ["<s>"] + sentence + ["</s>"]
            for i in range(len(tokens)):
                self.unigram_counts[tokens[i]] += 1
                self.vocab.add(tokens[i])
                if i > 0:
                    self.bigram_counts[(tokens[i-1], tokens[i])] += 1

    def prob(self, word, prev_word):
        """P(word | prev_word) with add-k smoothing."""
        numerator = self.bigram_counts[(prev_word, word)] + self.k
        denominator = self.unigram_counts[prev_word] + self.k * len(self.vocab)
        return numerator / denominator

    def sentence_log_prob(self, sentence):
        """Compute log probability of a sentence."""
        tokens = ["<s>"] + sentence + ["</s>"]
        log_prob = 0.0
        for i in range(1, len(tokens)):
            p = self.prob(tokens[i], tokens[i-1])
            log_prob += math.log2(p)
        return log_prob

    def perplexity(self, sentence):
        """Compute perplexity of a sentence."""
        tokens = ["<s>"] + sentence + ["</s>"]
        N = len(tokens) - 1  # number of predicted tokens
        log_prob = self.sentence_log_prob(sentence)
        return 2 ** (-log_prob / N)


# --- Example usage ---
corpus = [
    ["the", "cat", "sat", "on", "the", "mat"],
    ["the", "cat", "ate", "the", "fish"],
    ["the", "dog", "sat", "on", "the", "bed"],
    ["a", "cat", "sat", "on", "a", "mat"],
]

model = BigramLanguageModel(k=0.01)
model.train(corpus)

# Score two sentences
s1 = ["the", "cat", "sat", "on", "the", "mat"]
s2 = ["mat", "the", "on", "sat", "cat", "the"]

print(f"Log P(s1) = {model.sentence_log_prob(s1):.4f}")
print(f"Log P(s2) = {model.sentence_log_prob(s2):.4f}")
print(f"Perplexity(s1) = {model.perplexity(s1):.2f}")
print(f"Perplexity(s2) = {model.perplexity(s2):.2f}")

# Show bigram probabilities
print("\nSample bigram probabilities:")
for w1, w2 in [("the", "cat"), ("cat", "sat"), ("the", "unicorn")]:
    print(f"  P({w2}|{w1}) = {model.prob(w2, w1):.6f}")
```

**Expected output:**
```
Log P(s1) = -14.xxxx   (higher = more likely)
Log P(s2) = -28.xxxx   (lower  = less likely)
Perplexity(s1) ≈ 4.xx  (lower  = better model fit)
Perplexity(s2) ≈ 30.xx (higher = worse model fit)

s1 is grammatical → higher probability, lower perplexity
s2 is scrambled   → lower probability, higher perplexity
```

---

## 9. Limitations of N-gram Models

```
┌────────────────────────────────────────────────────────────────────────┐
│                    LIMITATIONS OF N-GRAM MODELS                       │
├──────────────────────┬─────────────────────────────────────────────────┤
│  1. SPARSITY         │ Most n-grams never appear in training data.    │
│                      │ As n increases, sparsity grows exponentially:  │
│                      │ V^n possible n-grams for vocab size V.         │
│                      │ V=50k → 50k³ = 1.25×10¹⁴ possible trigrams!  │
├──────────────────────┼─────────────────────────────────────────────────┤
│  2. NO LONG-RANGE    │ A trigram cannot capture dependencies like:    │
│     DEPENDENCIES     │ "The cat that the dog chased [ran] away"       │
│                      │ (cat→ran spans 5 words, beyond trigram window) │
├──────────────────────┼─────────────────────────────────────────────────┤
│  3. STORAGE          │ Storing all n-gram counts requires enormous    │
│                      │ memory. Google's 1T 5-gram dataset: ~24 GB    │
│                      │ compressed. Grows with corpus and vocab size.  │
├──────────────────────┼─────────────────────────────────────────────────┤
│  4. NO GENERALIZATION│ "cat sat on mat" and "kitten sat on rug" are   │
│                      │ treated as completely independent — no notion  │
│                      │ of word similarity (unlike embeddings).        │
├──────────────────────┼─────────────────────────────────────────────────┤
│  5. FIXED CONTEXT    │ Cannot dynamically decide how much context     │
│                      │ is relevant. Always uses exactly (n-1) words.  │
└──────────────────────┴─────────────────────────────────────────────────┘
```

These limitations led to the development of **neural language models** (RNNs, LSTMs, Transformers), which learn distributed representations and can model long-range dependencies.

---

## 10. Summary Table

| Concept                 | Key Idea                                         | Formula / Rule                                          |
|-------------------------|--------------------------------------------------|---------------------------------------------------------|
| Language Model          | Assigns probability to word sequences            | P(w₁, w₂, ..., wₙ)                                    |
| Chain Rule              | Decompose joint into conditionals                | ∏ P(wᵢ \| w₁...wᵢ₋₁)                                  |
| Markov Assumption       | Limit history to last (n−1) words                | P(wₙ \| wₙ₋₂, wₙ₋₁) for trigram                       |
| Unigram                 | No context, words are independent                | P(w) = Count(w) / N                                    |
| Bigram                  | Condition on previous word                       | P(wₙ \| wₙ₋₁) = C(wₙ₋₁,wₙ) / C(wₙ₋₁)                |
| Trigram                 | Condition on two previous words                  | P(wₙ \| wₙ₋₂,wₙ₋₁) = C(wₙ₋₂,wₙ₋₁,wₙ) / C(wₙ₋₂,wₙ₋₁)|
| MLE                     | Estimate probabilities from counts               | Count-based ratio                                       |
| Laplace Smoothing       | Add 1 to all counts                              | (C + 1) / (N + V)                                      |
| Add-k Smoothing         | Add fractional k to counts                       | (C + k) / (N + kV)                                     |
| Kneser-Ney Smoothing    | Discount + continuation probability              | Best smoothing for LMs                                  |
| Log Probabilities       | Avoid underflow, turn products into sums         | log P(W) = Σ log P(wᵢ \| context)                      |

---

## 11. Quick Revision Questions

1. **Why can't we compute the exact conditional P(wₙ | w₁, ..., wₙ₋₁) directly for long sentences?** *(Answer: The number of possible histories grows exponentially — most will never be observed in any corpus.)*

2. **A bigram model sees "dog barked" 5 times and "dog" 20 times in training. With Laplace smoothing and V = 10,000, what is P(barked | dog)?** *(Answer: (5+1)/(20+10000) = 6/10020 ≈ 0.000599)*

3. **Why does Kneser-Ney smoothing assign a low continuation probability to "Francisco" even though it has a high raw count?** *(Answer: "Francisco" appears in very few distinct contexts — almost always after "San" — so its versatility is low.)*

4. **What is the Markov assumption, and why is it necessary for n-gram models?**

5. **Why do we use log probabilities instead of raw probabilities when computing sentence scores?**

6. **Name three limitations of n-gram models that neural language models address.**

---

> [← Previous: Neural Sequence Labeling](../06-Sequence-Labeling/06-neural-sequence-labeling.md) | [Next → Perplexity](./02-perplexity.md)
