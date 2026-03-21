# Chapter 2: Perplexity

> **Unit 4.1 — NLP / 07 – Language Models** | [← Previous: N-gram Language Models](./01-ngram-language-models.md) | [Next → Neural Language Models](./03-neural-language-models.md)

---

## 1. Overview

**Perplexity** is the standard intrinsic evaluation metric for language models. It measures how "surprised" a model is by a test sequence — a model that assigns high probability to real-world sentences will have **low perplexity**, indicating it predicts language well. Formally, perplexity is the inverse probability of the test set, normalized by the number of words. It can be interpreted as the **weighted average branching factor**: at each position in the sequence, perplexity tells you the effective number of equally likely words the model considers. A perplexity of 50 means the model is, on average, as confused as if it had to choose uniformly among 50 words at every step. Lower perplexity always means a better model — it is more confident and more accurate in its predictions.

---

## 2. Definition of Perplexity

Given a test sequence W = w₁ w₂ ... wₙ and a language model that assigns probability P(W), perplexity is defined as:

```
                                    -1/N
PP(W) = P(w₁, w₂, ..., wₙ)

        ┌                          ┐ -1/N
      = │ ∏ P(wᵢ | w₁, ..., wᵢ₋₁) │
        └ i=1 to N                 ┘

Equivalently, using log probabilities:

                   1    N
PP(W) = 2^( - ─── · Σ  log₂ P(wᵢ | w₁, ..., wᵢ₋₁) )
                   N   i=1
```

**Key properties:**
- PP ≥ 1 (a perfect model that always predicts the correct next word with P = 1 has PP = 1)
- Lower PP → better model
- PP is measured on **held-out test data**, never on training data

---

## 3. Intuition — Weighted Average Branching Factor

Perplexity tells you the **effective number of choices** the model considers at each step:

```
┌──────────────────────────────────────────────────────────────────────────┐
│                  BRANCHING FACTOR VISUALIZATION                         │
│                                                                         │
│  Perplexity = 3: At each step, the model is as uncertain as choosing   │
│                  among 3 equally likely words.                          │
│                                                                         │
│            the ──┬── cat     (p ≈ 0.33)                                │
│                  ├── dog     (p ≈ 0.33)                                │
│                  └── bird    (p ≈ 0.33)                                │
│                                                                         │
│  Perplexity = 50: Like choosing uniformly among 50 words each step.    │
│                                                                         │
│            the ──┬── cat                                                │
│                  ├── dog                                                │
│                  ├── house        ← 50 branches,                       │
│                  ├── very            each with p ≈ 0.02                 │
│                  ├── ...                                                │
│                  └── zebra                                              │
│                                                                         │
│  Perplexity = 1:  Perfect prediction — model always knows the answer.  │
│                                                                         │
│            the ──── cat      (p = 1.0)                                 │
│                                                                         │
├──────────────────────────────────────────────────────────────────────────┤
│  LOWER PERPLEXITY = FEWER EFFECTIVE CHOICES = BETTER MODEL              │
└──────────────────────────────────────────────────────────────────────────┘
```

If the vocabulary has V words and the model is a **uniform random** model (every word equally likely), then PP = V. Any useful language model should have perplexity **well below** V.

---

## 4. Relationship to Cross-Entropy

Perplexity is directly related to **cross-entropy** H, which measures the average number of bits needed to encode the test data under the model:

```
              1    N
H(W) = - ─── · Σ  log₂ P(wᵢ | w₁, ..., wᵢ₋₁)
              N   i=1

PP(W) = 2^H(W)
```

| Metric         | Meaning                                          | Unit   |
|----------------|--------------------------------------------------|--------|
| Cross-Entropy  | Avg. bits per word to encode test data            | bits   |
| Perplexity     | 2 raised to the cross-entropy                    | words  |

**Example:** If cross-entropy H = 5.7 bits/word, then PP = 2^5.7 ≈ 52. The model is as confused as if choosing among 52 equally likely words.

Cross-entropy is a more "additive" quantity (useful for training with gradient descent), while perplexity provides a more **interpretable** measure for comparing models.

---

## 5. Step-by-Step Worked Example

Let's compute the perplexity of the sentence **"the cat sat"** under a simple bigram model.

### 5.1 Setup — Bigram Probabilities from Training

Suppose our trained bigram model gives these probabilities:

```
P(the | <s>)     = 0.40
P(cat | the)     = 0.30
P(sat | cat)     = 0.50
P(</s> | sat)    = 0.25
```

### 5.2 Sentence Probability

```
Sentence with boundary tokens:  <s> the cat sat </s>

P(W) = P(the|<s>) · P(cat|the) · P(sat|cat) · P(</s>|sat)
     = 0.40       · 0.30       · 0.50       · 0.25
     = 0.015

Number of predicted tokens:  N = 4   (the, cat, sat, </s>)
```

### 5.3 Log Probability

```
log₂ P(W) = log₂(0.40) + log₂(0.30) + log₂(0.50) + log₂(0.25)

           = -1.322 + (-1.737) + (-1.000) + (-2.000)

           = -6.059
```

### 5.4 Cross-Entropy

```
H = -1/N · log₂ P(W) = -1/4 · (-6.059) = 1.515 bits/word
```

### 5.5 Perplexity

```
PP = 2^H = 2^1.515 ≈ 2.86

Interpretation: On average, the model is as uncertain as if it were
                choosing among ≈ 2.86 equally likely words at each step.
                This is quite good — the model is fairly confident.
```

### 5.6 Comparison — A Worse Model

Suppose a second model gives more uniform (less confident) probabilities:

```
P(the | <s>)     = 0.10
P(cat | the)     = 0.05
P(sat | cat)     = 0.10
P(</s> | sat)    = 0.20

log₂ P(W) = log₂(0.10) + log₂(0.05) + log₂(0.10) + log₂(0.20)
           = -3.322 + (-4.322) + (-3.322) + (-2.322)
           = -13.288

H = -1/4 · (-13.288) = 3.322 bits/word

PP = 2^3.322 ≈ 10.0
```

```
┌──────────────────────────────────────────────────────┐
│           MODEL COMPARISON BY PERPLEXITY             │
│                                                      │
│  Model A:  PP ≈  2.86  ← Better (lower)             │
│  Model B:  PP ≈ 10.00  ← Worse  (higher)            │
│                                                      │
│  Model A assigns higher probability to the test      │
│  sentence, so it has lower perplexity and is the     │
│  better model for this data.                         │
└──────────────────────────────────────────────────────┘
```

---

## 6. Comparing Language Models Using Perplexity

The standard procedure for comparing two language models:

```
┌─────────────────────────────────────────────────────────────────────┐
│                  MODEL COMPARISON PIPELINE                          │
│                                                                     │
│   Corpus ──► Split ──┬── Training Set (80%)  ──► Train Model A     │
│                      │                        ──► Train Model B     │
│                      │                                              │
│                      ├── Validation Set (10%) ──► Tune hyperparams  │
│                      │                                              │
│                      └── Test Set (10%) ──► Compute PP_A and PP_B   │
│                                                                     │
│   If PP_A < PP_B on test set → Model A is better                   │
│                                                                     │
│   IMPORTANT: Both models MUST use the SAME vocabulary              │
│              and the SAME test set for fair comparison.             │
└─────────────────────────────────────────────────────────────────────┘
```

### 6.1 Training vs Test Perplexity — Overfitting Detection

A model that memorizes training data will show a large gap between training and test perplexity:

```
                 PP
                 │
           200 ──┤                           ╭── Test PP (Model C)
                 │                      ╭───╯
           150 ──┤                 ╭───╯
                 │            ╭───╯
           100 ──┤       ╭───╯─────────────── Test PP (Model D)
                 │  ╭───╯
            50 ──┤──╯
                 │          ╭──────────────── Train PP (Model D)
            20 ──┤─────────╯
                 │ ╲
            10 ──┤  ╲_______________________ Train PP (Model C)
                 │
             1 ──┤
                 └───┬────┬────┬────┬────┬── Model complexity →
                    uni  bi  tri  4-gram 5-gram

  Model C: Low train PP, high test PP → OVERFITTING (large gap)
  Model D: Moderate train PP, moderate test PP → GOOD FIT (small gap)
```

**Rules of thumb:**
- If train PP ≪ test PP → model is **overfitting** (memorizing training data)
- If train PP ≈ test PP and both are high → model is **underfitting**
- If train PP ≈ test PP and both are low → model **generalizes well**

---

## 7. Python Implementation — Computing Perplexity

```python
import math
from collections import Counter

class BigramPerplexity:
    """Compute perplexity of a test corpus under a bigram model."""

    def __init__(self, k=0.01):
        self.k = k  # add-k smoothing parameter
        self.bigram_counts = Counter()
        self.unigram_counts = Counter()
        self.vocab = set()

    def train(self, corpus):
        """Train on a corpus (list of sentences, each a list of words)."""
        for sentence in corpus:
            tokens = ["<s>"] + sentence + ["</s>"]
            for i in range(len(tokens)):
                self.unigram_counts[tokens[i]] += 1
                self.vocab.add(tokens[i])
                if i > 0:
                    self.bigram_counts[(tokens[i-1], tokens[i])] += 1

    def log_prob(self, word, context):
        """Log₂ P(word | context) with add-k smoothing."""
        numerator = self.bigram_counts[(context, word)] + self.k
        denominator = self.unigram_counts[context] + self.k * len(self.vocab)
        return math.log2(numerator / denominator)

    def corpus_perplexity(self, test_corpus):
        """Compute perplexity over an entire test corpus."""
        total_log_prob = 0.0
        total_tokens = 0

        for sentence in test_corpus:
            tokens = ["<s>"] + sentence + ["</s>"]
            for i in range(1, len(tokens)):
                total_log_prob += self.log_prob(tokens[i], tokens[i-1])
                total_tokens += 1

        cross_entropy = -total_log_prob / total_tokens
        perplexity = 2 ** cross_entropy
        return perplexity

    def sentence_perplexity(self, sentence):
        """Compute perplexity for a single sentence."""
        return self.corpus_perplexity([sentence])


# --- Example: Compare two models ---
train_corpus = [
    ["the", "cat", "sat", "on", "the", "mat"],
    ["the", "cat", "ate", "the", "fish"],
    ["the", "dog", "sat", "on", "the", "bed"],
    ["a", "dog", "chased", "the", "cat"],
    ["a", "cat", "sat", "on", "a", "mat"],
]

test_corpus = [
    ["the", "cat", "sat", "on", "the", "bed"],
    ["a", "dog", "ate", "the", "fish"],
]

# Model 1: Bigram with light smoothing
model1 = BigramPerplexity(k=0.01)
model1.train(train_corpus)

# Model 2: Bigram with heavy smoothing (more uniform → higher PP)
model2 = BigramPerplexity(k=1.0)
model2.train(train_corpus)

pp1 = model1.corpus_perplexity(test_corpus)
pp2 = model2.corpus_perplexity(test_corpus)

print(f"Model 1 (k=0.01) test perplexity: {pp1:.2f}")
print(f"Model 2 (k=1.00) test perplexity: {pp2:.2f}")
print(f"\nBetter model: {'Model 1' if pp1 < pp2 else 'Model 2'} (lower PP)")

# Per-sentence breakdown
for sent in test_corpus:
    pp = model1.sentence_perplexity(sent)
    print(f"  PP(\"{' '.join(sent)}\") = {pp:.2f}")

# Training perplexity (to check for overfitting)
train_pp = model1.corpus_perplexity(train_corpus)
print(f"\nModel 1 train perplexity: {train_pp:.2f}")
print(f"Model 1 test  perplexity: {pp1:.2f}")
gap = pp1 - train_pp
print(f"Gap (test - train): {gap:.2f} {'← possible overfitting' if gap > 20 else '← reasonable'}")
```

**Expected output:**
```
Model 1 (k=0.01) test perplexity: ~4-6   (low  → good fit)
Model 2 (k=1.00) test perplexity: ~8-12  (high → too smooth)

Better model: Model 1 (lower PP)

Model 1 train perplexity < test perplexity  (expected: model fits
training data better than unseen data)
```

---

## 8. Typical Perplexity Values

| Model Type                      | Typical Test PP  | Notes                                          |
|---------------------------------|------------------|-------------------------------------------------|
| Uniform random (V = 50,000)     | 50,000           | No learning at all — baseline                   |
| Unigram                         | 950 – 1,200      | Only word frequency, no context                 |
| Bigram                          | 150 – 300        | One word of context                             |
| Trigram                         | 80 – 180         | Two words of context                            |
| Trigram + Kneser-Ney            | 50 – 120         | Best classical smoothing                        |
| Feed-forward neural LM          | 80 – 130         | Fixed context window with embeddings            |
| LSTM / GRU                      | 50 – 90          | Variable-length context                         |
| Transformer (GPT-2 small)       | 20 – 40          | Attention over long context                     |
| Large LMs (GPT-3, LLaMA)       | 8 – 20           | Billions of parameters, massive training data   |

> **Note:** These numbers are approximate and depend heavily on the dataset, vocabulary size, and preprocessing. The Wall Street Journal (WSJ) section of the Penn Treebank (PTB) is a commonly used benchmark, with V ≈ 10,000.

---

## 9. Caveats and Pitfalls

```
┌──────────────────────────────────────────────────────────────────────────┐
│                     PERPLEXITY CAVEATS                                   │
├─────────────────────────┬────────────────────────────────────────────────┤
│  1. VOCABULARY MUST     │ A model with V=10,000 and PP=80 is NOT       │
│     MATCH               │ comparable to one with V=50,000 and PP=120.  │
│                         │ Larger vocab → harder task → higher PP.      │
│                         │ Always compare with the SAME vocabulary.     │
├─────────────────────────┼────────────────────────────────────────────────┤
│  2. OOV HANDLING        │ Out-of-vocabulary words must be handled      │
│     MATTERS             │ identically (e.g., map to <UNK> token).     │
│                         │ Different OOV strategies make PP             │
│                         │ incomparable.                                │
├─────────────────────────┼────────────────────────────────────────────────┤
│  3. TOKENIZATION        │ Word-level vs. subword (BPE) vs. character   │
│     GRANULARITY         │ tokenization produce different PP values.    │
│                         │ Only compare models with same tokenization.  │
├─────────────────────────┼────────────────────────────────────────────────┤
│  4. INTRINSIC ≠         │ Lower PP doesn't always mean better          │
│     EXTRINSIC           │ performance on downstream tasks (MT, ASR).   │
│                         │ PP is necessary but not sufficient.          │
├─────────────────────────┼────────────────────────────────────────────────┤
│  5. DOMAIN MISMATCH     │ A model trained on news will have high PP    │
│     INFLATES PP         │ on medical text, even if the model is good.  │
│                         │ Always evaluate on in-domain test data.      │
└─────────────────────────┴────────────────────────────────────────────────┘
```

**Rule for fair comparison:** Two perplexity scores are only comparable when the models use the **same vocabulary**, the **same tokenization**, the **same test set**, and handle OOV words in the **same way**.

---

## 10. Summary Table

| Concept                    | Key Idea                                          | Formula / Rule                                       |
|----------------------------|---------------------------------------------------|------------------------------------------------------|
| Perplexity Definition      | Inverse probability normalized by length          | PP(W) = P(w₁...wₙ)^(-1/N)                           |
| Cross-Entropy Link         | PP is 2 raised to the cross-entropy               | PP = 2^H                                            |
| Cross-Entropy              | Avg. bits per word under the model                | H = -(1/N) Σ log₂ P(wᵢ \| context)                  |
| Branching Factor           | PP = effective number of equally likely choices    | Uniform model over V words → PP = V                 |
| Lower Is Better            | Lower PP = higher probability on test data        | Model A better than B if PP_A < PP_B                 |
| Overfitting Signal         | Large gap between train and test PP               | Train PP ≪ Test PP → overfitting                    |
| Fair Comparison            | Same vocab, tokenization, test set required        | Cannot compare PP across different vocabularies      |
| Typical Good PP            | Modern neural LMs achieve PP < 30                 | PTB benchmark, V ≈ 10k                              |

---

## 11. Quick Revision Questions

1. **A bigram model assigns P(W) = 0.001 to a 10-word test sentence (N = 10 predicted tokens). What is the perplexity?** *(Answer: PP = 0.001^(-1/10) = (10³)^(1/10) = 10^(3/10) = 10^0.3 ≈ 2.00. The model is as uncertain as choosing between 2 equally likely words.)*

2. **If Model A has cross-entropy H = 4.5 bits/word and Model B has H = 5.2 bits/word on the same test set, which model is better and what are their perplexities?** *(Answer: Model A is better (lower H). PP_A = 2^4.5 ≈ 22.6, PP_B = 2^5.2 ≈ 36.8.)*

3. **A model achieves perplexity 5 on training data and perplexity 150 on test data. What does this indicate?** *(Answer: Severe overfitting — the model has memorized the training data but fails to generalize to unseen text.)*

4. **Why is it invalid to compare the perplexity of a word-level model (V = 50,000) with a character-level model (V = 27)?** *(Answer: Character-level models predict more tokens per sentence and have a much smaller vocabulary, making the task structurally different. The PP numbers are not on the same scale.)*

5. **A uniform language model over a vocabulary of 10,000 words has what perplexity?** *(Answer: PP = 10,000. Every word is equally likely, so the model has the maximum possible confusion for that vocabulary size.)*

6. **Name two ways perplexity can be misleading when evaluating a language model for a downstream task like machine translation.** *(Answer: (1) Lower PP doesn't guarantee better BLEU scores — the metrics measure different things. (2) Domain mismatch between LM training data and MT test data can inflate PP without reflecting true model quality.)*

---

> [← Previous: N-gram Language Models](./01-ngram-language-models.md) | [Next → Neural Language Models](./03-neural-language-models.md)
