# Chapter 4.5 — Plagiarism Detection with Rabin-Karp

> **Unit 4: Rabin-Karp Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Plagiarism detection is one of the most practical applications of Rabin-Karp.
By hashing fixed-length chunks and comparing them across documents, we can
efficiently find copied content.

---

## 1. The Plagiarism Problem

```
  Given: Document A (source), Document B (suspect)
  Find:  Common substrings of length ≥ k

  Example:
  ┌─────────────────────────────────────────────┐
  │  Doc A: "The quick brown fox jumps over"    │
  │  Doc B: "A slow brown fox jumps near"       │
  │                                              │
  │  Common substring: "brown fox jumps"         │
  │  Length: 15  ≥  k = 5  → PLAGIARISM FLAG    │
  └─────────────────────────────────────────────┘
```

---

## 2. Approach: k-gram Fingerprinting

```
  Step 1: Break both documents into k-grams (substrings of length k)
  Step 2: Hash each k-gram using rolling hash
  Step 3: Compare hash sets to find common fingerprints
  Step 4: Verify matches

  Document A: "abcdefgh"   k = 4
  ───────────────────────
  k-grams: "abcd" "bcde" "cdef" "defg" "efgh"
  hashes:    h₁     h₂     h₃     h₄     h₅

  Document B: "xyzdefgi"   k = 4
  ───────────────────────
  k-grams: "xyzd" "yzde" "zdef" "defg" "efgi"
  hashes:    h₆     h₇     h₈     h₄     h₉

  Common: h₄ → "defg" appears in both! → MATCH
```

---

## 3. Architecture

```
  ┌──────────┐     ┌──────────────┐     ┌──────────────┐
  │ Document │────→│  Tokenizer   │────→│   K-gram     │
  │    A     │     │              │     │  Generator   │
  └──────────┘     └──────────────┘     └──────┬───────┘
                                               │
                                        ┌──────▼───────┐
                                        │ Rolling Hash │
                                        │  (Rabin-Karp)│
                                        └──────┬───────┘
                                               │
                                        ┌──────▼───────┐
                                        │  Hash Set A  │
                                        └──────┬───────┘
                                               │ Compare
  ┌──────────┐     ┌──────────────┐     ┌──────▼───────┐
  │ Document │────→│  Same steps  │────→│  Hash Set B  │
  │    B     │     │              │     └──────┬───────┘
  └──────────┘     └──────────────┘            │
                                        ┌──────▼───────┐
                                        │ Intersection │
                                        │  = Matches   │
                                        └──────────────┘
```

---

## 4. Winnowing: Selecting Representative Fingerprints

```
  Problem: Storing ALL k-gram hashes is expensive for large documents.
  Solution: WINNOWING — select only some hashes, deterministically.

  Winnowing Algorithm:
  ────────────────────
  1. Compute hashes for all k-grams: h₁, h₂, h₃, ...
  2. Use a sliding WINDOW of size w
  3. In each window, select the MINIMUM hash
  4. Record selected hashes as fingerprints

  Hashes:  [12] [45] [7] [23] [56] [3] [89] [34]
  Window 1: [12  45   7]  → min = 7  ✓
  Window 2: [45   7  23]  → min = 7  (already selected)
  Window 3: [ 7  23  56]  → min = 7  (already selected)
  Window 4: [23  56   3]  → min = 3  ✓
  Window 5: [56   3  89]  → min = 3  (already selected)
  Window 6: [ 3  89  34]  → min = 3  (already selected)

  Fingerprints: {7, 3}   (much smaller than full set!)

  Guarantee: Any common substring of length ≥ k + w - 1
             will be detected.
```

---

## 5. Similarity Score

```
  After finding common fingerprints:

  Jaccard Similarity = |F_A ∩ F_B| / |F_A ∪ F_B|

  Example:
    Fingerprints of A: {3, 7, 12, 23, 56}
    Fingerprints of B: {3, 7, 34, 45, 89}
    
    Intersection: {3, 7}       → |∩| = 2
    Union: {3, 7, 12, 23, 34, 45, 56, 89}  → |∪| = 8

    Similarity = 2/8 = 25%

  ┌────────────────────────────────────────┐
  │ Similarity  │ Interpretation           │
  ├─────────────┼──────────────────────────┤
  │  0 - 10%    │ Likely original          │
  │ 10 - 30%    │ Some overlap (may be ok) │
  │ 30 - 60%    │ Suspicious               │
  │ 60 - 100%   │ Highly likely plagiarism │
  └─────────────┴──────────────────────────┘
```

---

## 6. Complete Algorithm

```
function DetectPlagiarism(docA, docB, k, threshold):
    // Step 1: Generate k-grams and hash
    hashesA = RollingHash(docA, k)    // set of all k-gram hashes
    hashesB = RollingHash(docB, k)

    // Step 2: Find common hashes
    common = hashesA ∩ hashesB

    // Step 3: Compute similarity
    similarity = |common| / |hashesA ∪ hashesB|

    // Step 4: Report
    if similarity > threshold:
        flag as plagiarism
        for each h in common:
            report matching positions

    return similarity
```

---

## 7. Implementation — Python

```python
def get_kgram_hashes(text, k, d=31, q=10**9 + 7):
    """Get all k-gram hashes with their positions using rolling hash."""
    n = len(text)
    if n < k:
        return {}
    
    # Precompute d^(k-1) mod q
    dk = 1
    for _ in range(k - 1):
        dk = (dk * d) % q
    
    # Hash first window
    h = 0
    for i in range(k):
        h = (h * d + ord(text[i])) % q
    
    hashes = {h: [0]}  # hash → list of positions
    
    # Roll
    for i in range(1, n - k + 1):
        h = (d * (h - ord(text[i - 1]) * dk) + ord(text[i + k - 1])) % q
        if h in hashes:
            hashes[h].append(i)
        else:
            hashes[h] = [i]
    
    return hashes


def detect_plagiarism(doc_a, doc_b, k=5):
    """Detect plagiarism between two documents."""
    hashes_a = get_kgram_hashes(doc_a, k)
    hashes_b = get_kgram_hashes(doc_b, k)
    
    set_a = set(hashes_a.keys())
    set_b = set(hashes_b.keys())
    
    common = set_a & set_b
    union = set_a | set_b
    
    similarity = len(common) / len(union) if union else 0
    
    # Find actual matching substrings
    matches = []
    for h in common:
        for pos_a in hashes_a[h]:
            substr_a = doc_a[pos_a:pos_a + k]
            for pos_b in hashes_b[h]:
                substr_b = doc_b[pos_b:pos_b + k]
                if substr_a == substr_b:  # verify (avoid spurious hits)
                    matches.append((substr_a, pos_a, pos_b))
    
    return similarity, matches


# Example
doc_a = "The quick brown fox jumps over the lazy dog"
doc_b = "A lazy brown fox jumps over the fence"

sim, matches = detect_plagiarism(doc_a, doc_b, k=5)
print(f"Similarity: {sim:.1%}")
for text, pa, pb in matches[:10]:
    print(f"  '{text}' at A[{pa}] and B[{pb}]")
```

---

## 8. Scaling to Many Documents

```
  For N documents (e.g., student submissions):

  Naive: Compare every pair → N(N-1)/2 comparisons

  Optimized with Rabin-Karp:
  ┌─────────────────────────────────────────────────┐
  │  1. Hash all k-grams in each document           │
  │  2. Build inverted index: hash → {doc IDs}      │
  │  3. For each hash appearing in ≥ 2 docs:        │
  │     Flag those document pairs                    │
  │  4. Only do detailed comparison on flagged pairs │
  └─────────────────────────────────────────────────┘

  Inverted Index:
    hash_42  →  {doc1, doc5, doc12}     ← docs 1,5,12 share a k-gram
    hash_73  →  {doc3}                  ← unique
    hash_99  →  {doc2, doc5}            ← docs 2,5 share a k-gram

  This is the foundation of tools like MOSS and Turnitin.
```

---

## 📝 Summary Table

| Concept | Details |
|---------|---------|
| k-gram | Substring of length k |
| Fingerprinting | Hashing k-grams with rolling hash |
| Winnowing | Selecting representative min-hashes from windows |
| Similarity | Jaccard = |A ∩ B| / |A ∪ B| |
| Time complexity | O(n₁ + n₂) for two documents |
| Scaling | Inverted index for many documents |
| Real tools | MOSS, Turnitin, Copyscape |

---

## ❓ Quick Revision Questions

1. **What is a k-gram in the context of plagiarism detection?**
   <details><summary>Answer</summary>A contiguous substring of length k extracted from the document. All k-grams form a sliding window over the text.</details>

2. **Why use rolling hash instead of hashing each k-gram independently?**
   <details><summary>Answer</summary>Rolling hash computes each subsequent k-gram hash in O(1) by reusing the previous hash, giving O(n) total instead of O(nk) with independent hashing.</details>

3. **What does winnowing achieve?**
   <details><summary>Answer</summary>It reduces the number of stored fingerprints by selecting only the minimum hash in each sliding window, while guaranteeing that any sufficiently long common substring will still be detected.</details>

4. **How is Jaccard similarity computed?**
   <details><summary>Answer</summary>Jaccard = |intersection of fingerprint sets| / |union of fingerprint sets|. A value closer to 1 indicates higher similarity.</details>

5. **How does an inverted index help with N documents?**
   <details><summary>Answer</summary>It maps each fingerprint hash to the set of documents containing it. Documents sharing fingerprints can be quickly identified without pairwise comparison, reducing work from O(N²) to near-linear.</details>

---

| [⬅️ Previous: Multiple Pattern Matching](04-multiple-pattern-matching.md) | [Next: Implementation ➡️](06-implementation.md) |
|:---|---:|
