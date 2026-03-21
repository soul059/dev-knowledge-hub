# TF-IDF (Term Frequency–Inverse Document Frequency)

## Overview

TF-IDF improves upon raw word counts by weighting words based on how **important** they are to a document relative to the entire corpus. Common words like "the" get low weight (they appear everywhere), while distinctive words like "neural" or "gradient" get high weight (they're specific to certain documents).

---

## The Math

```
TF-IDF(t, d, D) = TF(t, d) × IDF(t, D)

Where:
  TF(t, d)  = Term Frequency    = count(t in d) / total_words_in_d
  IDF(t, D) = Inverse Doc Freq  = log(N / df(t))
  
  t = term (word)
  d = document
  D = corpus (all documents)
  N = total number of documents
  df(t) = number of documents containing term t
```

### Worked Example

```
Corpus (3 documents):
  D1: "cat sat mat"
  D2: "cat dog"
  D3: "dog bird fish"

TF for "cat" in D1: 1/3 = 0.333
IDF for "cat": log(3/2) = 0.405     (appears in 2 of 3 docs)
TF-IDF("cat", D1) = 0.333 × 0.405 = 0.135

TF for "fish" in D3: 1/3 = 0.333
IDF for "fish": log(3/1) = 1.099    (appears in only 1 doc)
TF-IDF("fish", D3) = 0.333 × 1.099 = 0.366

"fish" gets HIGHER TF-IDF because it's more distinctive.
```

---

## Implementation

```python
from sklearn.feature_extraction.text import TfidfVectorizer

corpus = [
    "the cat sat on the mat",
    "the dog sat on the log",
    "the cat chased the dog around the park"
]

tfidf = TfidfVectorizer()
X = tfidf.fit_transform(corpus)

# Show feature names and their TF-IDF values
import pandas as pd
df = pd.DataFrame(X.toarray(), 
                  columns=tfidf.get_feature_names_out(),
                  index=['Doc1', 'Doc2', 'Doc3'])
print(df.round(3))
```
```
      around   cat  chased   dog   log   mat    on  park   sat   the
Doc1   0.000  0.36   0.000  0.00  0.00  0.47  0.36  0.0  0.36  0.61
Doc2   0.000  0.00   0.000  0.36  0.47  0.00  0.36  0.0  0.36  0.61
Doc3   0.378  0.29   0.378  0.29  0.00  0.00  0.00  0.378 0.00  0.49
```

Notice: "the" has low TF-IDF (appears in all docs), while "mat", "log", "park" have high values (unique to one doc).

---

## TF-IDF Variants

| Variant | TF Formula | When to Use |
|---------|-----------|-------------|
| Raw count | count(t,d) | Simple baseline |
| Term frequency | count(t,d) / len(d) | Normalize for doc length |
| Log-normalized | 1 + log(count(t,d)) | Reduce impact of very frequent terms |
| Boolean | 1 if t in d, else 0 | Presence matters, not frequency |
| Sublinear TF | 1 + log(tf) | Sklearn default with `sublinear_tf=True` |

---

## BoW vs TF-IDF

```
Document about machine learning in a general corpus:

Word "the":     BoW = high count,  TF-IDF = LOW   (in every doc)
Word "neural":  BoW = low count,   TF-IDF = HIGH  (distinctive!)

TF-IDF automatically downweights common words and
upweights distinctive terms — acting like a smart stopword filter.
```

| Feature | BoW | TF-IDF |
|---------|-----|--------|
| Values | Integer counts | Float weights |
| Common words | High values | Low values (downweighted) |
| Rare words | Low values | High values (upweighted) |
| Stopword handling | Need explicit removal | Automatically downweighted |
| Use case | Simple counting tasks | Classification, search, clustering |

---

## Practical Tips

```python
tfidf = TfidfVectorizer(
    max_features=10000,    # limit vocabulary
    min_df=5,              # ignore very rare words
    max_df=0.7,            # ignore words in >70% of docs
    ngram_range=(1, 2),    # include bigrams
    sublinear_tf=True,     # use log(1 + tf)
    norm='l2'              # L2 normalization (default)
)
```

---

## Revision Questions

1. **Explain the intuition behind IDF — why does it downweight common words?**
2. **Calculate TF-IDF by hand for a word that appears 3 times in a 10-word document, present in 2 of 100 documents.**
3. **Why does TF-IDF often outperform raw BoW for text classification?**
4. **What does `sublinear_tf=True` do and when is it useful?**
5. **How does the `ngram_range` parameter extend TF-IDF beyond single words?**

---

[Previous: 01-bag-of-words.md](01-bag-of-words.md) | [Next: 03-n-grams.md](03-n-grams.md)
