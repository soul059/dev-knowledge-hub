# Feature Extraction for Text Classification

## Overview

Feature extraction transforms raw text into numerical vectors that machine learning models can process. The quality of features directly determines classifier performance — garbage features in, garbage predictions out. This chapter covers the main feature extraction approaches from simple counts to learned representations.

---

## Feature Extraction Methods

```
┌──────────────────────────────────────────────────────┐
│           FEATURE EXTRACTION SPECTRUM                 │
│                                                      │
│  Simple ◀──────────────────────────────▶ Complex     │
│                                                      │
│  BoW → TF-IDF → N-grams → Embeddings → Transformers │
│  │       │         │          │              │       │
│  Sparse  Sparse    Sparse     Dense         Dense    │
│  Fast    Fast      Medium     Fast          Slow     │
│  Weak    Good      Better     Good          Best     │
└──────────────────────────────────────────────────────┘
```

---

## 1. Bag of Words Features

```python
from sklearn.feature_extraction.text import CountVectorizer

corpus = ["I love NLP", "NLP is great", "I love machine learning"]
vec = CountVectorizer()
X = vec.fit_transform(corpus)
print(vec.get_feature_names_out())
# ['great', 'is', 'learning', 'love', 'machine', 'nlp']
```

## 2. TF-IDF Features

```python
from sklearn.feature_extraction.text import TfidfVectorizer

tfidf = TfidfVectorizer(
    max_features=10000,
    ngram_range=(1, 2),    # unigrams + bigrams
    min_df=2,
    max_df=0.95,
    sublinear_tf=True
)
X = tfidf.fit_transform(corpus)
```

## 3. Handcrafted Features

```python
def extract_features(text):
    return {
        'word_count': len(text.split()),
        'char_count': len(text),
        'avg_word_length': sum(len(w) for w in text.split()) / len(text.split()),
        'exclamation_count': text.count('!'),
        'question_count': text.count('?'),
        'uppercase_ratio': sum(1 for c in text if c.isupper()) / len(text),
        'has_url': int('http' in text),
        'has_emoji': int(any(ord(c) > 127462 for c in text)),
    }
```

## 4. Word Embedding Features

```python
import numpy as np

def doc_to_embedding(text, word_vectors, dim=300):
    """Average word embeddings for a document."""
    words = text.lower().split()
    vectors = [word_vectors[w] for w in words if w in word_vectors]
    if vectors:
        return np.mean(vectors, axis=0)
    return np.zeros(dim)
```

## 5. Transformer Features (Modern)

```python
from transformers import AutoTokenizer, AutoModel
import torch

tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
model = AutoModel.from_pretrained("bert-base-uncased")

def get_bert_features(text):
    inputs = tokenizer(text, return_tensors="pt", truncation=True, max_length=512)
    with torch.no_grad():
        outputs = model(**inputs)
    # Use [CLS] token as document representation
    return outputs.last_hidden_state[:, 0, :].numpy()  # (1, 768)
```

---

## Feature Method Comparison

| Method | Dimensions | Semantic? | Speed | Best Accuracy | Use When |
|--------|:-:|:-:|:-:|:-:|---|
| BoW | V (large) | ❌ | ⚡ | Low | Quick baseline |
| TF-IDF | V (large) | ❌ | ⚡ | Good | Classical ML |
| TF-IDF + n-grams | V² (huge) | Partial | ⚡ | Better | Best classical approach |
| Avg. embeddings | 300 | ✅ | ⚡ | Good | When you have embeddings |
| BERT [CLS] | 768 | ✅ | 🐢 | Best | When accuracy matters |

---

## Combining Features

```python
from scipy.sparse import hstack
import numpy as np

# Combine TF-IDF with handcrafted features
tfidf_features = tfidf.fit_transform(texts)          # sparse
handcrafted = np.array([extract_features(t) for t in texts])  # dense

# Stack horizontally
from scipy.sparse import csr_matrix
combined = hstack([tfidf_features, csr_matrix(handcrafted)])
```

---

## Revision Questions

1. **What is the trade-off between TF-IDF and word embedding features?**
2. **Why might combining TF-IDF with handcrafted features outperform either alone?**
3. **How do you create a document-level vector from word embeddings?**
4. **What does the BERT [CLS] token represent and why is it used for classification?**
5. **When would handcrafted features (word count, punctuation) be valuable?**

---

[Previous: 01-problem-formulation.md](01-problem-formulation.md) | [Next: 03-traditional-ml.md](03-traditional-ml.md)
