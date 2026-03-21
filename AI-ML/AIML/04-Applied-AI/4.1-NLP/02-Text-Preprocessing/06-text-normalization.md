# Text Normalization

## Overview

Text normalization transforms text into a consistent, standard form. Beyond lowercasing and stemming, this includes handling Unicode, contractions, numbers, dates, abbreviations, and special characters. Proper normalization ensures that semantically identical inputs are represented the same way.

---

## Normalization Steps

```
Raw Input: "I'll meet u @ the café on Jan. 3rd, 2024!!! 😊"
    │
    ├─ Expand contractions    → "I will meet u at the café on Jan. 3rd, 2024!!! 😊"
    ├─ Expand abbreviations   → "I will meet you at the café on January 3rd, 2024!!! 😊"
    ├─ Unicode normalization   → "I will meet you at the cafe on January 3rd, 2024!!! 😊"
    ├─ Number normalization    → "I will meet you at the cafe on January third, 2024!!! 😊"
    ├─ Remove excess punct     → "I will meet you at the cafe on January third, 2024! 😊"
    └─ Handle emojis           → "I will meet you at the cafe on January third, 2024!"
```

---

## 1. Unicode Normalization

```python
import unicodedata

# Different Unicode representations of the same character
s1 = "café"           # 'é' as single character (NFC)
s2 = "cafe\u0301"     # 'e' + combining acute accent (NFD)

print(s1 == s2)        # False (different byte sequences!)
print(len(s1), len(s2))  # 4, 5

# Normalize to same form
s1_nfc = unicodedata.normalize('NFC', s1)
s2_nfc = unicodedata.normalize('NFC', s2)
print(s1_nfc == s2_nfc)  # True

# Strip accents entirely
def strip_accents(text):
    nfkd = unicodedata.normalize('NFKD', text)
    return ''.join(c for c in nfkd if not unicodedata.combining(c))

print(strip_accents("café résumé naïve"))  # "cafe resume naive"
```

---

## 2. Contraction Expansion

```python
contractions = {
    "i'm": "i am", "i'll": "i will", "i've": "i have",
    "i'd": "i would", "isn't": "is not", "aren't": "are not",
    "wasn't": "was not", "weren't": "were not",
    "don't": "do not", "doesn't": "does not",
    "didn't": "did not", "won't": "will not",
    "wouldn't": "would not", "couldn't": "could not",
    "shouldn't": "should not", "can't": "cannot",
    "it's": "it is", "that's": "that is",
    "let's": "let us", "who's": "who is",
    "what's": "what is", "there's": "there is",
    "he's": "he is", "she's": "she is",
    "they're": "they are", "we're": "we are",
    "you're": "you are", "you'll": "you will",
    "they've": "they have", "we've": "we have",
}

def expand_contractions(text):
    words = text.lower().split()
    return ' '.join(contractions.get(w, w) for w in words)

print(expand_contractions("I can't believe they're here"))
# "i cannot believe they are here"
```

---

## 3. Number and Date Handling

```python
import re

text = "I bought 3 items for $45.99 on 01/15/2024"

# Strategy 1: Replace all numbers with <NUM>
normalized = re.sub(r'\d+\.?\d*', '<NUM>', text)
# "I bought <NUM> items for $<NUM> on <NUM>/<NUM>/<NUM>"

# Strategy 2: Spell out small numbers
def normalize_numbers(text):
    number_words = {
        '0': 'zero', '1': 'one', '2': 'two', '3': 'three',
        '4': 'four', '5': 'five', '6': 'six', '7': 'seven',
        '8': 'eight', '9': 'nine', '10': 'ten'
    }
    words = text.split()
    return ' '.join(number_words.get(w, w) for w in words)
```

---

## 4. Handling Special Characters and Punctuation

```python
import re

text = "Hello!!! How are you??? I'm great... #NLP @user http://example.com"

# Remove URLs
text = re.sub(r'http\S+|www\.\S+', '', text)

# Remove mentions and hashtags
text = re.sub(r'[@#]\w+', '', text)

# Normalize repeated punctuation
text = re.sub(r'([!?.]){2,}', r'\1', text)

# Remove special characters (keep alphanumeric and basic punct)
text = re.sub(r"[^a-zA-Z0-9\s.,!?']", '', text)

print(text.strip())
# "Hello! How are you? I'm great."
```

---

## 5. Emoji Handling

```python
import re

text = "Great movie! 😊👍 Loved it 🎬"

# Option 1: Remove emojis
emoji_pattern = re.compile(
    "[\U0001F600-\U0001F64F"  # emoticons
    "\U0001F300-\U0001F5FF"   # symbols
    "\U0001F680-\U0001F6FF"   # transport
    "\U0001F1E0-\U0001F1FF"   # flags
    "]+", flags=re.UNICODE)

clean = emoji_pattern.sub('', text)
print(clean)  # "Great movie!  Loved it "

# Option 2: Replace with text descriptions (using emoji library)
# pip install emoji
# import emoji
# emoji.demojize("😊") → ":smiling_face_with_smiling_eyes:"
```

---

## Complete Normalization Pipeline

```python
import re
import unicodedata

def normalize_text(text):
    # 1. Unicode normalization
    text = unicodedata.normalize('NFKC', text)
    
    # 2. Lowercase
    text = text.lower()
    
    # 3. Expand contractions
    for contraction, expansion in contractions.items():
        text = text.replace(contraction, expansion)
    
    # 4. Remove URLs
    text = re.sub(r'http\S+', '', text)
    
    # 5. Remove HTML tags
    text = re.sub(r'<[^>]+>', '', text)
    
    # 6. Normalize repeated chars ("soooo" → "soo")
    text = re.sub(r'(.)\1{2,}', r'\1\1', text)
    
    # 7. Normalize whitespace
    text = re.sub(r'\s+', ' ', text).strip()
    
    return text

raw = "I can't believe   it's SOOOO good!!! 😊 <br/>"
print(normalize_text(raw))
# "i cannot believe it is soo good! 😊"
```

---

## Revision Questions

1. **Why might two visually identical strings compare as unequal, and how does Unicode normalization fix this?**
2. **When should you preserve contractions instead of expanding them?**
3. **What are the trade-offs between removing numbers vs replacing them with `<NUM>`?**
4. **How would you normalize social media text differently from formal documents?**
5. **Why is the order of normalization steps important?**

---

[Previous: 05-lemmatization.md](05-lemmatization.md) | [Next: 07-regular-expressions.md](07-regular-expressions.md)
