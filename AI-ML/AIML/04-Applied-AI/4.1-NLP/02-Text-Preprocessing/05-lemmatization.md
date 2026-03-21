# Lemmatization

## Overview

Lemmatization reduces words to their **lemma** — the base dictionary form. Unlike stemming, lemmatization uses vocabulary and morphological analysis to produce valid words: "better" → "good", "ran" → "run", "mice" → "mouse". It's more accurate but slower than stemming.

---

## Stemming vs Lemmatization

```
Word          Stemmer Output    Lemmatizer Output
─────────────────────────────────────────────────
"better"      "better"          "good"        ← knows irregular form
"studies"     "studi"           "study"       ← valid word
"ran"         "ran"             "run"         ← knows past tense
"wolves"      "wolv"            "wolf"        ← knows irregular plural
"happily"     "happili"         "happily"     ← adverb (or "happy" with POS)
"caring"      "care"            "care"        ← both work here
```

---

## NLTK WordNet Lemmatizer

```python
from nltk.stem import WordNetLemmatizer
import nltk
nltk.download('wordnet', quiet=True)

lemmatizer = WordNetLemmatizer()

# Default: assumes NOUN
print(lemmatizer.lemmatize("cats"))       # cat
print(lemmatizer.lemmatize("running"))    # running  ← wrong! (treated as noun)
print(lemmatizer.lemmatize("better"))     # better   ← wrong! (treated as noun)

# With POS tag: much better results
print(lemmatizer.lemmatize("running", pos='v'))  # run     ✓
print(lemmatizer.lemmatize("better", pos='a'))   # good    ✓
print(lemmatizer.lemmatize("ran", pos='v'))      # run     ✓
print(lemmatizer.lemmatize("geese", pos='n'))    # goose   ✓
```

### POS-Aware Lemmatization

```python
from nltk import pos_tag, word_tokenize
from nltk.corpus import wordnet

def get_wordnet_pos(treebank_tag):
    """Convert Penn Treebank POS to WordNet POS."""
    if treebank_tag.startswith('J'):
        return wordnet.ADJ
    elif treebank_tag.startswith('V'):
        return wordnet.VERB
    elif treebank_tag.startswith('N'):
        return wordnet.NOUN
    elif treebank_tag.startswith('R'):
        return wordnet.ADV
    return wordnet.NOUN  # default

text = "The striped bats are hanging on their feet for best results"
tokens = word_tokenize(text)
tagged = pos_tag(tokens)

lemmatizer = WordNetLemmatizer()
lemmas = [lemmatizer.lemmatize(word, get_wordnet_pos(tag)) 
          for word, tag in tagged]

print(f"Original:    {tokens}")
print(f"Lemmatized:  {lemmas}")
# Original:    ['The', 'striped', 'bats', 'are', 'hanging', 'on', 'their', 'feet', 'for', 'best', 'results']
# Lemmatized:  ['The', 'strip', 'bat', 'be', 'hang', 'on', 'their', 'foot', 'for', 'good', 'result']
```

---

## spaCy Lemmatizer

```python
import spacy
nlp = spacy.load("en_core_web_sm")

doc = nlp("The children were running happily in the gardens")
for token in doc:
    if token.lemma_ != token.text.lower():
        print(f"  {token.text:12s} → {token.lemma_:10s} (POS: {token.pos_})")
# children     → child      (POS: NOUN)
# were         → be         (POS: AUX)
# running      → run        (POS: VERB)
# gardens      → garden     (POS: NOUN)
```

spaCy handles POS tagging automatically — no manual POS mapping needed.

---

## Comparison Table

| Feature | Stemming | Lemmatization |
|---------|----------|---------------|
| Output | Root/stem (may not be a word) | Dictionary lemma (valid word) |
| Method | Rule-based suffix stripping | Morphological analysis + dictionary |
| Speed | Fast | Slower |
| Accuracy | Lower (over/under-stemming) | Higher |
| POS needed | No | Yes (for best results) |
| "better" | "better" | "good" |
| "ran" | "ran" | "run" |
| Libraries | NLTK Porter/Snowball | NLTK WordNet, spaCy |
| Use case | Search, IR, quick baseline | Classification, understanding tasks |

---

## When to Use Lemmatization

```
✅ Use lemmatization when:
  • You need valid dictionary words
  • Downstream task is classification or similarity
  • Working with features humans will inspect
  • Language has rich morphology

❌ Stick with stemming when:
  • Speed is critical (large-scale IR)
  • Approximate matching is sufficient
  • Using it as input to TF-IDF for basic models

🚫 Skip both when:
  • Using subword tokenization (BPE/WordPiece)
  • Using pre-trained transformers (BERT, GPT)
```

---

## Revision Questions

1. **What is the key difference between stemming and lemmatization?**
2. **Why does the WordNet lemmatizer need POS tags to work correctly?**
3. **Show what happens when you lemmatize "running" as a noun vs as a verb.**
4. **Why is spaCy's lemmatizer easier to use than NLTK's for beginners?**
5. **When should you skip both stemming and lemmatization entirely?**

---

[Previous: 04-stemming.md](04-stemming.md) | [Next: 06-text-normalization.md](06-text-normalization.md)
