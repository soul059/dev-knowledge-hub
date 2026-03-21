# Stopword Removal

## Overview

Stopwords are high-frequency words that carry little semantic meaning on their own — words like "the", "is", "at", "which", "on". Removing them reduces noise and feature dimensionality, letting models focus on content words. However, stopword removal can harm tasks where function words carry meaning (e.g., sentiment: "not good" → removing "not" changes meaning entirely).

---

## What Are Stopwords?

```
Sentence: "The cat is sitting on the mat in the room"

Content words:  cat, sitting, mat, room  → carry meaning
Stopwords:      the, is, on, the, in, the → structural glue

After removal:  "cat sitting mat room"
  → Core meaning preserved, noise reduced
```

---

## Standard Stopword Lists

```python
from nltk.corpus import stopwords
import nltk
nltk.download('stopwords', quiet=True)

# NLTK English stopwords (179 words)
stop_words = set(stopwords.words('english'))
print(f"Count: {len(stop_words)}")
print(sorted(list(stop_words))[:20])
# ['a', 'about', 'above', 'after', 'again', 'against', 'ain', 'all',
#  'am', 'an', 'and', 'any', 'are', 'aren', "aren't", 'as', 'at',
#  'be', 'because', 'been']
```

```python
import spacy
nlp = spacy.load("en_core_web_sm")

# spaCy stopwords (326 words)
print(f"Count: {len(nlp.Defaults.stop_words)}")
# Includes: a, about, above, after, again, against, all, also, am, ...
```

---

## Implementation

```python
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords

stop_words = set(stopwords.words('english'))

text = "This is a very important document about machine learning"
tokens = word_tokenize(text.lower())

# Remove stopwords
filtered = [w for w in tokens if w not in stop_words and w.isalpha()]
print(f"Before: {tokens}")
print(f"After:  {filtered}")
# Before: ['this', 'is', 'a', 'very', 'important', 'document', 'about', 'machine', 'learning']
# After:  ['important', 'document', 'machine', 'learning']
```

---

## When NOT to Remove Stopwords

| Task | Why Keep Stopwords | Example |
|------|-------------------|---------|
| Sentiment Analysis | Negation words matter | "not good" → removing "not" = disaster |
| Machine Translation | Every word matters | "the" maps to articles in other languages |
| Question Answering | "who", "what", "where" are stopwords | "Who is the president?" |
| Language Modeling | Predict next word needs all words | Perplexity calculation |
| Transformers/BERT | Model handles importance internally | Attention learns what to ignore |
| Phrase detection | "to be or not to be" | Removing stopwords destroys the phrase |

```
"I do not like this movie at all"

With stopwords removed: "like movie"      → POSITIVE? ✗ WRONG
With stopwords kept:    "do not like"     → NEGATIVE  ✓ CORRECT
```

---

## Custom Stopword Lists

```python
# Domain-specific stopword customization
default_stops = set(stopwords.words('english'))

# For sentiment analysis: remove negation words from stoplist
sentiment_stops = default_stops - {'not', 'no', 'nor', 'neither', 
                                     'never', 'nobody', 'nothing',
                                     'hardly', 'barely', "doesn't",
                                     "isn't", "wasn't", "shouldn't",
                                     "wouldn't", "couldn't", "won't"}

# For medical NLP: add domain-specific stopwords
medical_stops = default_stops | {'patient', 'doctor', 'hospital', 
                                   'medical', 'clinical'}
```

---

## Impact on Feature Space

```
Corpus: 10,000 documents, 50,000 unique words

After stopword removal:
  Vocabulary: ~49,800 words (removed ~200 stopwords)
  But total tokens reduced by ~40-50%!
  
  TF-IDF matrix: much sparser, more discriminative features
```

---

## Revision Questions

1. **What are stopwords and why are they removed?**
2. **Give an example where removing stopwords changes the meaning of a sentence.**
3. **Why should you NOT remove stopwords when using BERT or GPT?**
4. **How would you customize a stopword list for sentiment analysis?**
5. **How does stopword removal affect TF-IDF feature matrices?**

---

[Previous: 02-lowercasing.md](02-lowercasing.md) | [Next: 04-stemming.md](04-stemming.md)
