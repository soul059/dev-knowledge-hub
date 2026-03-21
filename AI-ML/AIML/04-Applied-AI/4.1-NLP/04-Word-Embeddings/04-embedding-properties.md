# Word Embedding Properties

## Overview

Word embeddings capture rich linguistic properties — semantic similarity, analogical relationships, clustering by topic, and even syntactic patterns. Understanding these properties helps you evaluate embedding quality, debug models, and choose the right embeddings for your task.

---

## 1. Semantic Similarity

Words with similar meanings have similar vectors:

```python
import gensim.downloader as api
model = api.load("glove-wiki-gigaword-100")

# Semantic similarity (cosine)
pairs = [
    ("happy", "joyful"),      # synonyms
    ("king", "queen"),        # related
    ("cat", "dog"),           # same category  
    ("cat", "democracy"),     # unrelated
]
for w1, w2 in pairs:
    sim = model.similarity(w1, w2)
    print(f"  {w1:12s} - {w2:12s} : {sim:.3f}")
# happy        - joyful       : 0.725
# king         - queen        : 0.751
# cat          - dog          : 0.802
# cat          - democracy    : 0.089
```

---

## 2. Analogical Reasoning

Embeddings capture **relational patterns** through vector arithmetic:

```
king - man + woman ≈ queen     (gender)
Paris - France + Japan ≈ Tokyo  (capital)
walking - walked + swam ≈ swimming  (tense)
bigger - big + small ≈ smaller  (comparative)
```

```python
# A is to B as C is to ?
def analogy(model, a, b, c):
    result = model.most_similar(positive=[b, c], negative=[a], topn=1)
    return result[0]

print(analogy(model, "man", "king", "woman"))     # ('queen', 0.73)
print(analogy(model, "france", "paris", "japan"))  # ('tokyo', 0.72)
```

---

## 3. Clustering

Similar words naturally cluster together in embedding space:

```
┌──────────────────────────────────────────┐
│          Embedding Space (2D PCA)         │
│                                          │
│    dog •  cat •                           │
│      horse •     Animals                  │
│         fish •                            │
│                                          │
│                     • Paris              │
│          • Berlin    • London  Cities    │
│            • Tokyo                       │
│                                          │
│  • happy  • joyful                       │
│    • glad    Emotions                    │
│      • sad  • angry                      │
│                                          │
└──────────────────────────────────────────┘
```

---

## 4. Linear Substructures

Embeddings organize concepts along linear directions:

```
Gender direction:    man ←────────────→ woman
                     king ←───────────→ queen
                     uncle ←──────────→ aunt

Tense direction:     walk ←───────────→ walked
                     run ←────────────→ ran
                     eat ←────────────→ ate

Plurality direction: cat ←────────────→ cats
                     dog ←────────────→ dogs
```

---

## 5. Known Biases

```
Historical bias in embeddings:
  "doctor" - "man" + "woman" ≈ "nurse"    ← gender bias
  "programmer" - "man" + "woman" ≈ "homemaker"  ← harmful!
  
Embeddings trained on biased text learn and amplify biases.

Mitigation approaches:
  • Debiasing algorithms (Bolukbasi et al., 2016)
  • Balanced training corpora
  • Post-hoc projection to remove bias directions
  • Evaluation with bias benchmarks (WEAT, SEAT)
```

---

## Evaluating Embedding Quality

| Evaluation Type | Method | Datasets |
|----------------|--------|----------|
| **Intrinsic** | Word similarity correlation | WordSim-353, SimLex-999 |
| **Intrinsic** | Analogy accuracy | Google analogy dataset |
| **Extrinsic** | Downstream task performance | Sentiment, NER, QA |

```python
# Intrinsic evaluation: analogy accuracy
from gensim.models import KeyedVectors

model = KeyedVectors.load_word2vec_format('vectors.bin', binary=True)
accuracy = model.evaluate_word_analogies('questions-words.txt')
print(f"Analogy accuracy: {accuracy[0]:.1%}")
```

---

## Summary of Properties

| Property | Description | Example |
|----------|-------------|---------|
| Semantic similarity | Similar words → close vectors | cat ≈ dog |
| Analogies | Relationships preserved as directions | king-man+woman=queen |
| Clustering | Topics form natural groups | Animals cluster together |
| Linear substructures | Concepts vary along directions | Gender, tense, plurality |
| Compositionality | Sum of word vectors ≈ phrase meaning | "machine"+"learning" |
| Bias reflection | Reflect training data biases | Gender/racial stereotypes |

---

## Revision Questions

1. **What does it mean for word embeddings to capture "semantic similarity"?**
2. **How do word analogies work mathematically in embedding space?**
3. **What types of biases can word embeddings learn, and how can they be mitigated?**
4. **What is the difference between intrinsic and extrinsic evaluation of embeddings?**
5. **Why do words naturally cluster in embedding space?**

---

[Previous: 03-fasttext.md](03-fasttext.md) | [Next: 05-embedding-visualization.md](05-embedding-visualization.md)
