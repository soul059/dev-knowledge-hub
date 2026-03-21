# Tokenization

## Overview

Tokenization is the process of splitting text into smaller meaningful units called **tokens**. It is the first and most critical step in any NLP pipeline — every downstream task depends on good tokenization. Tokens can be words, subwords, characters, or sentences depending on the approach.

---

## Types of Tokenization

```
Input: "I can't believe it's raining!"

Word-level:     ["I", "can't", "believe", "it's", "raining", "!"]
  or:           ["I", "ca", "n't", "believe", "it", "'s", "raining", "!"]

Subword (BPE):  ["I", "can", "'t", "believe", "it", "'s", "rain", "##ing", "!"]

Character:      ["I", " ", "c", "a", "n", "'", "t", " ", "b", ...]

Sentence:       ["I can't believe it's raining!"]
```

---

## Word Tokenization

### Simple Split

```python
text = "Hello world! How are you?"
tokens = text.split()
print(tokens)  # ['Hello', 'world!', 'How', 'are', 'you?']
# Problem: punctuation attached to words
```

### NLTK Word Tokenizer

```python
from nltk.tokenize import word_tokenize
import nltk
nltk.download('punkt_tab', quiet=True)

text = "Dr. Smith doesn't like Mr. Jones's idea."
tokens = word_tokenize(text)
print(tokens)
# ['Dr.', 'Smith', 'does', "n't", 'like', 'Mr.', 'Jones', "'s", 'idea', '.']
# ✓ Handles contractions: doesn't → does + n't
# ✓ Handles abbreviations: Dr. stays together
# ✓ Separates punctuation
```

### spaCy Tokenizer

```python
import spacy
nlp = spacy.load("en_core_web_sm")

doc = nlp("I'll meet you at the U.S. Bank at 3:30 p.m.")
tokens = [token.text for token in doc]
print(tokens)
# ['I', "'ll", 'meet', 'you', 'at', 'the', 'U.S.', 'Bank', 'at', '3:30', 'p.m.']
# ✓ Handles abbreviations, times, contractions
```

---

## Sentence Tokenization

```python
from nltk.tokenize import sent_tokenize

text = """Dr. Smith went to Washington. He arrived at 3 p.m. 
The meeting was productive. "It went well," he said."""

sentences = sent_tokenize(text)
for i, s in enumerate(sentences):
    print(f"  {i+1}. {s}")
# 1. Dr. Smith went to Washington.
# 2. He arrived at 3 p.m.
# 3. The meeting was productive.
# 4. "It went well," he said.
# ✓ Doesn't split on "Dr." or "p.m."
```

---

## Subword Tokenization

Modern NLP uses subword tokenization to handle the open-vocabulary problem.

### Byte Pair Encoding (BPE)

```
Vocabulary building (simplified):

Start: ['l', 'o', 'w', 'e', 'r', 'n', 'w', 's', 't']

Step 1: Most frequent pair ('l','o') → merge to 'lo'
Step 2: Most frequent pair ('lo','w') → merge to 'low'
Step 3: Most frequent pair ('e','r') → merge to 'er'
Step 4: Most frequent pair ('low','er') → merge to 'lower'
...

Result: Common words stay whole, rare words split:
  "lower"     → ["lower"]          (common — single token)
  "lowest"    → ["low", "est"]     (decomposed)
  "lowering"  → ["lower", "ing"]   (decomposed)
  "xyzabc"    → ["x", "y", "z", "a", "b", "c"]  (unknown — character fallback)
```

### WordPiece (used by BERT)

```python
from transformers import BertTokenizer

tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')

text = "unbelievably"
tokens = tokenizer.tokenize(text)
print(tokens)  # ['un', '##believ', '##ably']
# "##" prefix means "continuation of previous token"

text2 = "transformers are revolutionary"
tokens2 = tokenizer.tokenize(text2)
print(tokens2)  # ['transform', '##ers', 'are', 'revolutionary']
```

### SentencePiece (used by T5, LLaMA)

```python
# Language-agnostic, works on raw text including spaces
# Treats the input as a sequence of Unicode characters
# No pre-tokenization needed

# "▁" represents a space character
# "Hello World" → ["▁Hello", "▁World"]
```

---

## Tokenization Comparison

| Method | Vocabulary Size | OOV Handling | Used By |
|--------|:-:|:-:|---------|
| Word-level | 100K+ | ❌ `<UNK>` token | NLTK, spaCy |
| Character-level | ~256 | ✅ Never OOV | Older models |
| BPE | 30K–50K | ✅ Subword decomposition | GPT-2, GPT-3 |
| WordPiece | 30K | ✅ `##` continuation | BERT |
| SentencePiece | 32K | ✅ Language-agnostic | T5, LLaMA |
| Unigram | 32K | ✅ Probabilistic | XLNet, ALBERT |

---

## Edge Cases and Challenges

```
Contractions:    "don't"  → "do" + "n't"  OR  "don't"?
Hyphenated:      "state-of-the-art" → 1 token or 4?
URLs:            "https://example.com/path" → how to split?
Emojis:          "😊" → keep? remove? encode?
Numbers:         "3.14" → "3" + "." + "14"?
Code:            "my_variable_name" → camelCase? underscores?
CJK languages:   No spaces between words (Chinese, Japanese)
```

---

## Revision Questions

1. **What is the difference between word, subword, and character tokenization?**
2. **Why does simple `split()` fail as a tokenizer? Give examples.**
3. **Explain how BPE builds its vocabulary from training data.**
4. **What does the `##` prefix mean in WordPiece tokenization?**
5. **Why is subword tokenization preferred in modern NLP models?**
6. **How does SentencePiece handle languages without spaces?**

---

[Previous: Unit 1 - 05-text-vs-language.md](../01-NLP-Fundamentals/05-text-vs-language.md) | [Next: 02-lowercasing.md](02-lowercasing.md)
