# Bag of Words (BoW)

## Overview

Bag of Words is one of the simplest text representation methods. It represents a document as a vector of word counts, completely ignoring grammar and word order. Despite its simplicity, BoW forms the foundation for many NLP tasks and is still useful for baseline models.

---

## How BoW Works

```
Document 1: "the cat sat on the mat"
Document 2: "the dog sat on the log"

Step 1: Build vocabulary from all documents
  Vocabulary: {cat, dog, log, mat, on, sat, the}

Step 2: Count word occurrences per document

         cat  dog  log  mat  on  sat  the
Doc 1: [  1    0    0    1    1   1    2  ]
Doc 2: [  0    1    1    0    1   1    2  ]
```

---

## Implementation

```python
from sklearn.feature_extraction.text import CountVectorizer

corpus = [
    "The cat sat on the mat",
    "The dog sat on the log",
    "The cat chased the dog"
]

vectorizer = CountVectorizer()
X = vectorizer.fit_transform(corpus)

print("Vocabulary:", vectorizer.get_feature_names_out())
print("\nBoW Matrix:")
print(X.toarray())
```
```
Vocabulary: ['cat' 'chased' 'dog' 'log' 'mat' 'on' 'sat' 'the']

BoW Matrix:
[[1 0 0 0 1 1 1 2]    ← "The cat sat on the mat"
 [0 0 1 1 0 1 1 2]    ← "The dog sat on the log"
 [1 1 1 0 0 0 0 2]]   ← "The cat chased the dog"
```

---

## Binary BoW

Instead of counts, just mark presence (1) or absence (0):

```python
vectorizer = CountVectorizer(binary=True)
X = vectorizer.fit_transform(corpus)
print(X.toarray())
# [[1 0 0 0 1 1 1 1]    ← "the" appears, binary = 1 (not 2)
#  [0 0 1 1 0 1 1 1]
#  [1 1 1 0 0 0 0 1]]
```

---

## What BoW Loses

```
"The dog bit the man"     → [1, 1, 0, 1, 1]  (bit, dog, man, the)
"The man bit the dog"     → [1, 1, 0, 1, 1]  (same vector!)

Word ORDER is completely lost — these have opposite meanings
but identical BoW representations.

Also lost:
  • Grammar and syntax
  • Context and meaning
  • Relationships between words
```

---

## Strengths and Weaknesses

| Aspect | Details |
|--------|---------|
| ✅ Simple | Easy to implement and understand |
| ✅ Fast | Efficient computation |
| ✅ Baseline | Good starting point for classification |
| ✅ Interpretable | Can inspect which words matter |
| ❌ No order | "dog bit man" = "man bit dog" |
| ❌ High dimensionality | Vocabulary can be 100K+ features |
| ❌ Sparse | Most entries are zero |
| ❌ No semantics | "happy" and "joyful" are completely unrelated |

---

## Practical Tips

```python
# Control vocabulary size
vectorizer = CountVectorizer(
    max_features=5000,     # keep top 5000 words
    min_df=2,              # ignore words appearing in < 2 docs
    max_df=0.95,           # ignore words in > 95% of docs
    stop_words='english'   # remove English stopwords
)
```

---

## Revision Questions

1. **What information does BoW discard when representing text?**
2. **What is the difference between count-based and binary BoW?**
3. **Why are BoW vectors typically very sparse?**
4. **How do `min_df` and `max_df` help control vocabulary size?**
5. **Give an example where BoW fails to distinguish two sentences with different meanings.**

---

[Previous: Unit 2 - 07-regular-expressions.md](../02-Text-Preprocessing/07-regular-expressions.md) | [Next: 02-tf-idf.md](02-tf-idf.md)
