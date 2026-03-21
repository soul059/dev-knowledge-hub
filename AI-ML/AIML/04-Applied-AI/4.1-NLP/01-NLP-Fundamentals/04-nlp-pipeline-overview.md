# NLP Pipeline Overview

## Overview

Every NLP system follows a pipeline — a sequence of processing steps that transform raw text into structured, actionable output. Understanding this pipeline is essential because each stage's quality directly impacts downstream performance.

---

## The Standard NLP Pipeline

```
┌────────────────────────────────────────────────────────────────────┐
│                     CLASSIC NLP PIPELINE                           │
│                                                                    │
│  Raw Text                                                          │
│     │                                                              │
│     ▼                                                              │
│  ┌──────────────────┐                                              │
│  │ 1. Text Cleanup  │  Remove HTML, fix encoding, normalize       │
│  └────────┬─────────┘                                              │
│           ▼                                                        │
│  ┌──────────────────┐                                              │
│  │ 2. Tokenization  │  Split into words/subwords/sentences        │
│  └────────┬─────────┘                                              │
│           ▼                                                        │
│  ┌──────────────────┐                                              │
│  │ 3. Normalization │  Lowercasing, stemming, lemmatization       │
│  └────────┬─────────┘                                              │
│           ▼                                                        │
│  ┌──────────────────┐                                              │
│  │ 4. Stopword      │  Remove common words (the, is, at)         │
│  │    Removal        │                                             │
│  └────────┬─────────┘                                              │
│           ▼                                                        │
│  ┌──────────────────┐                                              │
│  │ 5. Feature       │  BoW, TF-IDF, embeddings                   │
│  │    Extraction     │                                             │
│  └────────┬─────────┘                                              │
│           ▼                                                        │
│  ┌──────────────────┐                                              │
│  │ 6. Modeling      │  Classification, NER, translation, etc.     │
│  └────────┬─────────┘                                              │
│           ▼                                                        │
│  ┌──────────────────┐                                              │
│  │ 7. Post-process  │  Format output, filter, validate            │
│  └──────────────────┘                                              │
└────────────────────────────────────────────────────────────────────┘
```

---

## Modern Pipeline (Deep Learning Era)

With pre-trained transformer models, many classical steps are handled implicitly:

```
┌──────────────────────────────────────────────────────┐
│              MODERN NLP PIPELINE                      │
│                                                      │
│  Raw Text                                            │
│     │                                                │
│     ▼                                                │
│  ┌─────────────────┐                                 │
│  │ Subword          │  BPE / WordPiece / SentencePiece│
│  │ Tokenization     │  (handles OOV automatically)   │
│  └────────┬────────┘                                 │
│           ▼                                          │
│  ┌─────────────────┐                                 │
│  │ Pre-trained      │  BERT, GPT, T5                 │
│  │ Model            │  (contextual representations)  │
│  └────────┬────────┘                                 │
│           ▼                                          │
│  ┌─────────────────┐                                 │
│  │ Task Head        │  Classification, QA, NER, etc. │
│  │ (fine-tuned)     │                                │
│  └────────┬────────┘                                 │
│           ▼                                          │
│  Output                                              │
└──────────────────────────────────────────────────────┘

What's eliminated:
  ✗ Manual stopword removal (model learns what to ignore)
  ✗ Stemming/lemmatization (subword tokenization)
  ✗ Feature engineering (learned representations)
```

---

## Pipeline Stage Details

### Stage 1: Text Cleanup

```python
import re
import html

raw = "<p>Check out https://example.com! It's   great 😊</p>"

# Remove HTML tags
text = re.sub(r'<[^>]+>', '', raw)
# Decode HTML entities
text = html.unescape(text)
# Remove URLs
text = re.sub(r'https?://\S+', '', text)
# Normalize whitespace
text = re.sub(r'\s+', ' ', text).strip()

print(text)  # "Check out It's great 😊"
```

### Stage 2: Tokenization

```python
import nltk
from nltk.tokenize import word_tokenize, sent_tokenize

text = "Dr. Smith went to Washington. He arrived at 3 p.m."

sentences = sent_tokenize(text)
# ['Dr. Smith went to Washington.', 'He arrived at 3 p.m.']

tokens = word_tokenize(text)
# ['Dr.', 'Smith', 'went', 'to', 'Washington', '.', 'He', ...]
```

### Stage 3–4: Normalization & Stopword Removal

```python
from nltk.corpus import stopwords
from nltk.stem import WordNetLemmatizer

lemmatizer = WordNetLemmatizer()
stop_words = set(stopwords.words('english'))

tokens = ['The', 'cats', 'were', 'running', 'quickly']
processed = [
    lemmatizer.lemmatize(t.lower())
    for t in tokens
    if t.lower() not in stop_words
]
# ['cat', 'running', 'quickly']
```

### Stage 5: Feature Extraction

```python
from sklearn.feature_extraction.text import TfidfVectorizer

corpus = [
    "NLP is fascinating",
    "Machine learning powers NLP",
    "Deep learning transforms NLP"
]

vectorizer = TfidfVectorizer()
X = vectorizer.fit_transform(corpus)
print(f"Shape: {X.shape}")  # (3, 8) — 3 docs, 8 features
print(f"Features: {vectorizer.get_feature_names_out()}")
```

---

## Classic vs Modern Pipeline Comparison

| Aspect | Classic Pipeline | Modern Pipeline |
|--------|-----------------|-----------------|
| Tokenization | Word-level (NLTK, spaCy) | Subword (BPE, WordPiece) |
| Normalization | Stemming, lemmatization | Handled by model |
| Features | Hand-crafted (TF-IDF) | Learned embeddings |
| Context | Limited (n-gram window) | Full sequence (attention) |
| OOV handling | Dictionary lookup | Subword decomposition |
| Setup effort | High (many steps) | Low (tokenizer + model) |
| Domain adaptation | Feature re-engineering | Fine-tuning |
| Compute cost | Low | High (GPU needed) |

---

## Choosing the Right Pipeline

```
Small dataset (<1K samples)?
  └─ YES → Classic pipeline (TF-IDF + SVM)
  └─ NO  → How much compute?
              ├─ Limited → Distilled model (DistilBERT)
              └─ Ample  → Full transformer (BERT, GPT)
```

---

## Revision Questions

1. **List the seven stages of a classic NLP pipeline in order.**
2. **Which classic pipeline stages are eliminated by modern transformer models, and why?**
3. **What is the role of the feature extraction stage?**
4. **When might you prefer a classic NLP pipeline over a modern deep learning one?**
5. **How does subword tokenization solve the out-of-vocabulary problem?**

---

[Previous: 03-challenges-in-nlp.md](03-challenges-in-nlp.md) | [Next: 05-text-vs-language.md](05-text-vs-language.md)
