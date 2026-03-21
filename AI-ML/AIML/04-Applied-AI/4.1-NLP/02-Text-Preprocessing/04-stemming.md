# Stemming

## Overview

Stemming reduces words to their **stem** (root form) by stripping suffixes using heuristic rules. The goal is to group related word forms together: "running", "runs", "ran" → "run". Unlike lemmatization, stemming doesn't produce valid dictionary words — it just chops off endings.

---

## How Stemming Works

```
┌──────────────────────────────────────────────┐
│              STEMMING PROCESS                 │
│                                              │
│  "running"  ──strip "ning"──▶  "run"    ✓   │
│  "studies"  ──strip "ies"+y──▶ "studi"  ✗   │
│  "flying"   ──strip "ing"──▶   "fli"    ✗   │
│  "happiness"──strip "ness"──▶  "happi"  ✗   │
│  "caresses" ──strip "es"──▶    "caress" ✓   │
│                                              │
│  Note: stems are NOT always real words!       │
└──────────────────────────────────────────────┘
```

---

## Porter Stemmer (Most Common)

The Porter Stemmer (1980) applies a series of rules in sequence:

```python
from nltk.stem import PorterStemmer

stemmer = PorterStemmer()

words = ["running", "runs", "ran", "runner", 
         "studies", "studying", "studied",
         "happily", "happiness", "happy",
         "connection", "connected", "connecting"]

for word in words:
    print(f"  {word:15s} → {stemmer.stem(word)}")
```
```
  running         → run
  runs            → run
  ran             → ran
  runner          → runner
  studies         → studi
  studying        → studi
  studied         → studi
  happily         → happili
  happiness       → happi
  happy           → happi
  connection      → connect
  connected       → connect
  connecting      → connect
```

### Porter Stemmer Rules (Simplified)

```
Step 1: Plurals and past participles
  SSES → SS    (caresses → caress)
  IES  → I     (ponies → poni)
  S    → ∅     (cats → cat)  [if stem has vowel]

Step 2: Common suffixes
  ATIONAL → ATE  (relational → relate)
  IZER    → IZE  (digitizer → digitize)
  FULNESS → FUL  (hopefulness → hopeful)

Step 3-5: More suffix stripping
  ICATE → IC   (triplicate → triplic)
  AL    → ∅    (revival → reviv)
  ...
```

---

## Snowball Stemmer (Improved Porter)

```python
from nltk.stem import SnowballStemmer

# Available languages
print(SnowballStemmer.languages)
# ('arabic', 'danish', 'dutch', 'english', 'finnish', 'french', 
#  'german', 'hungarian', 'italian', 'norwegian', 'porter', 
#  'portuguese', 'romanian', 'russian', 'spanish', 'swedish')

stemmer = SnowballStemmer("english")

words = ["generously", "generous", "generation", "generate"]
for word in words:
    print(f"  {word:15s} → {stemmer.stem(word)}")
# generously      → generous
# generous        → generous
# generation      → generat
# generate        → generat
```

---

## Lancaster Stemmer (Aggressive)

```python
from nltk.stem import LancasterStemmer

lancaster = LancasterStemmer()
porter = PorterStemmer()

words = ["maximum", "presumably", "multiply", "provision"]
print(f"{'Word':15s} {'Porter':12s} {'Lancaster':12s}")
print("-" * 39)
for w in words:
    print(f"{w:15s} {porter.stem(w):12s} {lancaster.stem(w):12s}")
```
```
Word            Porter       Lancaster   
---------------------------------------
maximum         maximum      maxim       
presumably      presum       presum      
multiply        multipli     multiply    
provision       provis       provid      
```

---

## Stemmer Comparison

| Stemmer | Speed | Aggressiveness | Quality | Languages |
|---------|:-----:|:-:|:-:|:-:|
| Porter | Fast | Medium | Good | English only |
| Snowball | Fast | Medium | Better | 15+ languages |
| Lancaster | Fastest | Very aggressive | Over-stems | English only |

```
Over-stemming:  "university" and "universe" → both stem to "univers"
                (they're unrelated — bad!)

Under-stemming: "alumni" and "alumnus" → different stems
                (they're related — missed!)
```

---

## When to Use Stemming

```
✅ Good for:
  • Information retrieval / search engines
  • Bag-of-Words / TF-IDF features
  • When speed matters more than precision
  • Quick baseline models

❌ Avoid when:
  • Part-of-speech matters (stem loses POS info)
  • Need valid dictionary words (use lemmatization)
  • Using modern embeddings/transformers (handle this internally)
  • Language with complex morphology (Arabic, Turkish)
```

---

## Revision Questions

1. **What is the difference between a stem and a lemma?**
2. **Why does the Porter Stemmer sometimes produce non-words?**
3. **Explain over-stemming and under-stemming with examples.**
4. **When would you choose Lancaster over Snowball stemmer?**
5. **Why is stemming unnecessary when using transformer models?**

---

[Previous: 03-stopword-removal.md](03-stopword-removal.md) | [Next: 05-lemmatization.md](05-lemmatization.md)
