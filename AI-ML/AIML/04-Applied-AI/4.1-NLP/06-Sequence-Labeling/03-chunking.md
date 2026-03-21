# Chunking (Shallow Parsing)

## Overview

Chunking groups consecutive tokens into **phrases** (chunks) without building a full parse tree. It identifies noun phrases (NP), verb phrases (VP), and prepositional phrases (PP) — providing a middle ground between POS tagging and full syntactic parsing.

---

## What Chunking Does

```
Sentence: "The big brown dog chased the small cat in the park"

POS tags:  DT  JJ   JJ    NN  VBD   DT   JJ   NN  IN DT  NN

Chunks:
  [The big brown dog]  [chased]  [the small cat]  [in]  [the park]
        NP                VP          NP            PP      NP

NP = Noun Phrase     → "the big brown dog"
VP = Verb Phrase     → "chased"
PP = Prepositional   → "in the park"
```

---

## Implementation with NLTK

```python
import nltk
from nltk import pos_tag, word_tokenize, RegexpParser

text = "The black cat sat on the wooden mat"
tokens = word_tokenize(text)
tagged = pos_tag(tokens)

# Define chunk grammar
grammar = r"""
    NP: {<DT>?<JJ>*<NN.*>+}    # Optional determiner, any adjectives, nouns
    VP: {<VB.*>+}               # One or more verbs
    PP: {<IN><NP>}              # Preposition followed by NP
"""

parser = RegexpParser(grammar)
tree = parser.parse(tagged)

# Print chunk tree
tree.pretty_print()
```

```
                   S
    ┌──────┬───┬──┴──────────────┐
    NP     VP  PP                NP
  ┌─┼──┐  │  ┌┴┐          ┌────┼───┐
 The black cat sat on     the wooden mat
```

---

## Chunking vs Full Parsing

| Feature | Chunking | Full Parsing |
|---------|----------|-------------|
| Depth | Flat (one level) | Recursive tree |
| Speed | Fast | Slow |
| Accuracy | High | Lower |
| Output | Non-overlapping phrases | Full constituency/dependency tree |
| Use case | IE, quick NP extraction | Grammar analysis, MT |

---

## Practical Use: Extracting Noun Phrases

```python
import spacy
nlp = spacy.load("en_core_web_sm")

text = "Machine learning algorithms achieve state-of-the-art performance on NLP tasks"
doc = nlp(text)

print("Noun phrases:")
for chunk in doc.noun_chunks:
    print(f"  '{chunk.text}' (root: {chunk.root.text}, dep: {chunk.root.dep_})")
# 'Machine learning algorithms' (root: algorithms, dep: nsubj)
# 'state-of-the-art performance' (root: performance, dep: dobj)
# 'NLP tasks' (root: tasks, dep: pobj)
```

---

## Revision Questions

1. **What is chunking and how does it differ from full parsing?**
2. **What are the common chunk types (NP, VP, PP)?**
3. **Write a regex grammar to extract noun phrases.**
4. **Why is chunking useful for information extraction?**
5. **How does spaCy's `noun_chunks` relate to chunking?**

---

[Previous: 02-named-entity-recognition.md](02-named-entity-recognition.md) | [Next: 04-bio-tagging.md](04-bio-tagging.md)
