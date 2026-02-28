# Chapter 2: Sentence Similarity (LeetCode 737)

## Overview

**Sentence Similarity II** asks whether two sentences are similar given a list of similar word pairs, where similarity is **transitive**: if "great" ~ "fine" and "fine" ~ "good", then "great" ~ "good". This is a direct application of Union-Find.

---

## Problem Statement

```
Input:
  sentence1 = ["great", "acting", "skills"]
  sentence2 = ["fine", "drama", "talent"]
  pairs = [["great","fine"], ["acting","drama"], 
           ["skills","talent"]]

Output: true

Because:
  great ~ fine (direct)
  acting ~ drama (direct) 
  skills ~ talent (direct)
  
Input:
  sentence1 = ["great", "acting", "skills"]
  sentence2 = ["fine", "drama", "talent"]
  pairs = [["great","fine"], ["fine","good"], 
           ["acting","drama"], ["skills","talent"]]

Output: true
  (great ~ fine ~ good, so great ~ good by transitivity)
```

---

## Why Union-Find?

```
Similar word pairs define an EQUIVALENCE RELATION:
  - Reflexive: word ~ word (always)
  - Symmetric: if A ~ B then B ~ A
  - Transitive: if A ~ B and B ~ C then A ~ C

Union-Find is PERFECT for equivalence classes!

  Each pair (A, B) → Union(A, B)
  Check sentence: for each position i,
    connected(word1[i], word2[i])?
```

---

## Solution

```python
def areSentencesSimilarTwo(sentence1, sentence2, pairs):
    if len(sentence1) != len(sentence2):
        return False
    
    # Union-Find with string keys (using dictionary)
    parent = {}
    
    def find(x):
        if x not in parent:
            parent[x] = x
        if parent[x] != x:
            parent[x] = find(parent[x])
        return parent[x]
    
    def union(x, y):
        parent[find(x)] = find(y)
    
    # Build equivalence classes from pairs
    for a, b in pairs:
        union(a, b)
    
    # Check each word pair
    for w1, w2 in zip(sentence1, sentence2):
        if w1 == w2:
            continue
        if find(w1) != find(w2):
            return False
    
    return True
```

---

## Trace Walkthrough

```
pairs = [("great","fine"), ("fine","good"), 
         ("acting","drama"), ("skills","talent")]

Processing pairs:
  union("great", "fine"):
    parent = {great: fine, fine: fine}
    
  union("fine", "good"):
    find(fine) = fine
    find(good) = good  (newly added)
    parent = {great: fine, fine: good, good: good}
    
  union("acting", "drama"):
    parent = {..., acting: drama, drama: drama}
    
  union("skills", "talent"):
    parent = {..., skills: talent, talent: talent}

Now checking: sentence1 = ["great","acting","skills"]
              sentence2 = ["fine","drama","talent"]

  great vs fine:
    find(great) → parent[great]=fine → parent[fine]=good → good
    find(fine) → parent[fine]=good → good
    Same root → ✓

  acting vs drama:
    find(acting) → drama
    find(drama) → drama
    Same root → ✓

  skills vs talent:
    find(skills) → talent
    find(talent) → talent
    Same root → ✓

Result: true ✓
```

---

## Comparison: DFS vs Union-Find

```
DFS/BFS Approach:
  Build graph: word → [similar words]
  For each word pair: BFS from word1 to see if word2 reachable
  Time: O(P + N × P) where P = pairs, N = sentence length
  
Union-Find Approach:
  Build: O(P × α(P))
  Check: O(N × α(P))
  Time: O((P + N) × α(P)) ≈ O(P + N)

Union-Find is simpler and faster!

┌──────────────┬───────────┬──────────────────┐
│ Approach     │ Build     │ Per Query        │
├──────────────┼───────────┼──────────────────┤
│ DFS/BFS      │ O(P)      │ O(P) per word    │
│ Union-Find   │ O(P×α)    │ O(α) per word    │
└──────────────┴───────────┴──────────────────┘
```

---

## Sentence Similarity I (LeetCode 734) - Non-Transitive

```
If similarity is NOT transitive (only direct pairs):
  → Use a HashSet of pairs
  → No Union-Find needed!

def areSentencesSimilar(s1, s2, pairs):
    if len(s1) != len(s2):
        return False
    pair_set = set()
    for a, b in pairs:
        pair_set.add((a, b))
        pair_set.add((b, a))
    
    for w1, w2 in zip(s1, s2):
        if w1 != w2 and (w1, w2) not in pair_set:
            return False
    return True

Key insight: 
  Transitive? → Union-Find
  Non-transitive? → HashSet
```

---

## Variations

```
1. Sentence Similarity with Groups (k-way):
   Given word groups (each group = synonyms)
   → Union all words in each group

2. Multi-language Translation:
   word_en ~ word_fr ~ word_de (transitive)
   Check if two words "mean the same thing"
   → Union-Find on words

3. Weighted Similarity:
   Each pair has a score, query: 
   "what's the similarity between x and y?"
   → Weighted Union-Find (multiply along path)
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem | Check if sentences are similar |
| Similarity type | Transitive (equivalence relation) |
| DSU nodes | Words (strings) |
| Union trigger | Given similar pair |
| Query | Same root for corresponding words |
| Time complexity | O(P + N) effectively |
| Key insight | Transitivity = equivalence = Union-Find |

---

## Quick Revision Questions

1. **Why is Union-Find appropriate for transitive similarity?**
2. **How do you handle string keys in Union-Find?**
3. **What's the difference between Sentence Similarity I and II?**
4. **What's the time complexity advantage over DFS?**
5. **What edge case must you check before comparing words?**

---

[← Previous: Accounts Merge](01-accounts-merge.md) | [Next: Regions Cut by Slashes →](03-regions-cut-by-slashes.md)
