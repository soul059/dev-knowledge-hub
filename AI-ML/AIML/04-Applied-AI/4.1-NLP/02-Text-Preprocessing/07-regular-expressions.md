# Regular Expressions for Text

## Overview

Regular expressions (regex) are powerful pattern-matching tools essential for NLP text processing. They enable you to find, extract, replace, and validate text patterns efficiently. Every NLP practitioner should be fluent in regex for data cleaning, feature extraction, and text manipulation.

---

## Regex Fundamentals

```
Pattern     Matches                   Example
──────────────────────────────────────────────────
.           Any single character      "c.t" → cat, cut, c9t
\d          Any digit [0-9]           "\d{3}" → 123, 456
\w          Word char [a-zA-Z0-9_]   "\w+" → hello, var_1
\s          Whitespace                "hello\sworld"
\b          Word boundary             "\bcat\b" → "cat" not "catch"
^           Start of string           "^Hello"
$           End of string             "world$"
*           0 or more                 "ab*c" → ac, abc, abbc
+           1 or more                 "ab+c" → abc, abbc (not ac)
?           0 or 1                    "colou?r" → color, colour
{n,m}       n to m repetitions        "\d{2,4}" → 12, 123, 1234
[abc]       Character class           "[aeiou]" → any vowel
[^abc]      Negated class             "[^0-9]" → any non-digit
(...)       Capture group             "(\d+)-(\d+)" → groups
|           Alternation (OR)          "cat|dog"
```

---

## NLP-Specific Regex Patterns

### Email Extraction
```python
import re

text = "Contact us at info@company.com or support@example.org"
emails = re.findall(r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}', text)
print(emails)  # ['info@company.com', 'support@example.org']
```

### URL Extraction
```python
text = "Visit https://www.example.com or http://blog.site.org/post"
urls = re.findall(r'https?://[^\s<>"{}|\\^`\[\]]+', text)
print(urls)  # ['https://www.example.com', 'http://blog.site.org/post']
```

### Phone Numbers
```python
text = "Call 555-123-4567 or (555) 987-6543 or +1-555-000-1111"
phones = re.findall(r'[\+]?[\d\-\(\)\s]{7,15}', text)
# Clean up: ['555-123-4567', '(555) 987-6543', '+1-555-000-1111']
```

### Hashtags and Mentions
```python
text = "Great talk by @elonmusk about #AI and #MachineLearning"
hashtags = re.findall(r'#\w+', text)    # ['#AI', '#MachineLearning']
mentions = re.findall(r'@\w+', text)    # ['@elonmusk']
```

---

## Text Cleaning with Regex

```python
import re

def clean_text(text):
    # Remove HTML tags
    text = re.sub(r'<[^>]+>', '', text)
    
    # Remove URLs
    text = re.sub(r'https?://\S+|www\.\S+', '[URL]', text)
    
    # Remove email addresses
    text = re.sub(r'\S+@\S+\.\S+', '[EMAIL]', text)
    
    # Normalize repeated characters: "sooooo" → "soo"
    text = re.sub(r'(.)\1{2,}', r'\1\1', text)
    
    # Normalize repeated punctuation: "!!!" → "!"
    text = re.sub(r'([!?.]){2,}', r'\1', text)
    
    # Remove extra whitespace
    text = re.sub(r'\s+', ' ', text).strip()
    
    return text

raw = "<p>OMG!!! Visit https://wow.com  sooooo   cool!!!</p>"
print(clean_text(raw))
# "OMG! Visit [URL] soo cool!"
```

---

## Named Groups for Information Extraction

```python
import re

# Extract structured data from unstructured text
pattern = r'(?P<name>[A-Z][a-z]+)\s+(?P<age>\d+)\s+years?\s+old'

text = "Alice 30 years old, Bob 25 year old, Charlie 35 years old"
for match in re.finditer(pattern, text):
    print(f"  Name: {match.group('name')}, Age: {match.group('age')}")
# Name: Alice, Age: 30
# Name: Bob, Age: 25
# Name: Charlie, Age: 35
```

---

## Regex for Tokenization

```python
import re

text = "Hello, world! It's a test-case for NLP (v2.0)."

# Simple word tokenization
tokens = re.findall(r"\b\w+(?:'\w+)?\b", text)
print(tokens)
# ['Hello', 'world', "It's", 'a', 'test', 'case', 'for', 'NLP', 'v2', '0']

# Keep contractions and hyphenated words
tokens = re.findall(r"\w+(?:[-']\w+)*", text)
print(tokens)
# ['Hello', 'world', "It's", 'a', 'test-case', 'for', 'NLP', 'v2', '0']
```

---

## Common Regex Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| Greedy matching | `<.*>` matches `<b>text</b>` entirely | Use `<.*?>` (lazy) |
| No word boundaries | `\bcat\b` vs `cat` (matches "concatenate") | Always use `\b` for words |
| Backtracking | Complex patterns can be extremely slow | Simplify, use atomic groups |
| Unicode | `\w` may not match accented chars | Use `re.UNICODE` flag |
| Multiline | `^` and `$` match string edges only | Use `re.MULTILINE` flag |

---

## Quick Reference for NLP

```python
# Most useful regex for NLP preprocessing:

re.sub(r'<[^>]+>', '', text)           # Strip HTML
re.sub(r'https?://\S+', '', text)      # Remove URLs
re.sub(r'[^\w\s]', '', text)           # Remove punctuation
re.sub(r'\d+', '<NUM>', text)          # Replace numbers
re.sub(r'\s+', ' ', text)             # Normalize whitespace
re.findall(r'#\w+', text)             # Extract hashtags
re.findall(r'\b[A-Z]{2,}\b', text)    # Find acronyms
re.sub(r'(.)\1{2,}', r'\1\1', text)   # Reduce repeated chars
```

---

## Revision Questions

1. **What is the difference between `*`, `+`, and `?` quantifiers?**
2. **Write a regex to extract all email addresses from a text.**
3. **What is the difference between greedy and lazy matching?**
4. **How would you use regex to normalize "sooooo goooood!!!" to "soo good!"?**
5. **Why should you use `\b` word boundaries when searching for specific words?**
6. **Write a regex to find all dates in MM/DD/YYYY format.**

---

[Previous: 06-text-normalization.md](06-text-normalization.md) | [Next: Unit 3 - 01-bag-of-words.md](../03-Text-Representation/01-bag-of-words.md)
