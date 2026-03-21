# Lowercasing

## Overview

Lowercasing converts all characters in text to their lowercase form. It's one of the simplest yet most impactful preprocessing steps — reducing vocabulary size and ensuring that "Apple", "apple", and "APPLE" are treated as the same token. However, lowercasing is not always appropriate; it can destroy important information.

---

## Why Lowercase?

```
Without lowercasing:                With lowercasing:
  "The"  → token_1                   "the"  → token_1
  "the"  → token_2                   "the"  → token_1  (same!)
  "THE"  → token_3                   "the"  → token_1  (same!)
  
Vocabulary reduced, better generalization.
```

---

## Implementation

```python
text = "Apple announced the new iPhone in New York City."

# Simple lowercasing
lower_text = text.lower()
print(lower_text)
# "apple announced the new iphone in new york city."

# With NLTK tokenization
from nltk.tokenize import word_tokenize
tokens = word_tokenize(text)
lower_tokens = [t.lower() for t in tokens]
print(lower_tokens)
# ['apple', 'announced', 'the', 'new', 'iphone', 'in', 'new', 'york', 'city', '.']
```

---

## When NOT to Lowercase

| Scenario | Why Lowercasing Hurts | Example |
|----------|----------------------|---------|
| Named Entity Recognition | Capitalization signals proper nouns | "Apple" (company) vs "apple" (fruit) |
| Sentiment (emphasis) | CAPS indicate strong emotion | "This is TERRIBLE" vs "terrible" |
| Acronyms | Lose meaning | "US" (country) vs "us" (pronoun) |
| Case-sensitive codes | Identifiers change meaning | "myVar" ≠ "myvar" in code |
| German nouns | All nouns are capitalized | "Liebe" (love) vs "liebe" (dear) |

```
"I love Apple products"    → NER needs: Apple = ORG
"I love apple juice"       → NER needs: apple = common noun

After lowercasing both become "apple" → distinction LOST
```

---

## Best Practices

```
Task                    Lowercase?
─────────────────────────────────
Text classification     ✅ Usually yes
Sentiment analysis      ⚠️ Depends (CAPS = emphasis)
Named Entity Recognition ❌ No — case is a feature
Machine Translation     ❌ No — preserve case
Search/Information Retrieval ✅ Yes (case-insensitive matching)
Modern Transformers     ⚠️ Model-specific:
                          BERT uncased → already lowercased
                          BERT cased  → preserves case
                          GPT models  → preserves case
```

---

## Revision Questions

1. **Why does lowercasing reduce vocabulary size? Give a numerical example.**
2. **In what NLP tasks should you avoid lowercasing?**
3. **What information is lost when lowercasing "US troops in the US"?**
4. **What is the difference between BERT-cased and BERT-uncased models?**
5. **How does lowercasing interact with subword tokenization?**

---

[Previous: 01-tokenization.md](01-tokenization.md) | [Next: 03-stopword-removal.md](03-stopword-removal.md)
