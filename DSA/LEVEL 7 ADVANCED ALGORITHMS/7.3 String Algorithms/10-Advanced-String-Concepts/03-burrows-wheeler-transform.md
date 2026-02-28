# Chapter 10.3 — Burrows-Wheeler Transform (BWT)

> **Unit 10: Advanced String Concepts** | [Course Home](../README.md)

---

## 📋 Chapter Overview

The **Burrows-Wheeler Transform** rearranges a string to group similar
characters together, making it highly compressible. It is the backbone
of tools like **bzip2** and is used in bioinformatics for sequence
alignment (BWA, Bowtie).

---

## 1. BWT Construction

```
  String: "banana$"

  Step 1: Generate ALL rotations
  ┌───────────────┐
  │ banana$       │  rotation 0
  │ anana$b       │  rotation 1
  │ nana$ba       │  rotation 2
  │ ana$ban       │  rotation 3
  │ na$bana       │  rotation 4
  │ a$banan       │  rotation 5
  │ $banana       │  rotation 6
  └───────────────┘

  Step 2: Sort rotations lexicographically
  ┌───────────────┐
  │ $banana       │  ← sorted[0]
  │ a$banan       │  ← sorted[1]
  │ ana$ban       │  ← sorted[2]
  │ anana$b       │  ← sorted[3]
  │ banana$       │  ← sorted[4]
  │ na$bana       │  ← sorted[5]
  │ nana$ba       │  ← sorted[6]
  └───────────────┘

  Step 3: BWT = last column
  BWT = "annb$aa"

  ┌────────────────────────────────────────────────────┐
  │  First column F: $aaabnn  (sorted characters)      │
  │  Last column  L: annb$aa  (BWT output)             │
  │                                                     │
  │  Notice: L groups similar characters!              │
  │  "aa" appears together, "nn" appears together.     │
  │  This makes L highly compressible.                 │
  └────────────────────────────────────────────────────┘
```

---

## 2. Why BWT is Compressible

```
  Original:   "banana$"  → characters scattered
  BWT:        "annb$aa"  → characters clustered!

  Run-Length Encoding:
    "annb$aa" → a(1) n(2) b(1) $(1) a(2) = shorter!

  For real-world text (English, DNA):
  ┌────────────────────────────────────────────────────┐
  │  If 'e' appears at position i,                     │
  │  the character BEFORE 'e' in the original text     │
  │  is often 'h' (as in "the", "he", "she").         │
  │                                                     │
  │  BWT groups these context characters together!     │
  │  Many 'h's cluster → great for compression.       │
  └────────────────────────────────────────────────────┘

  bzip2 pipeline:
    Text → BWT → Move-to-front → Run-length → Huffman/Arithmetic
```

---

## 3. Inverse BWT (Reconstruction)

```
  Given only BWT "annb$aa", reconstruct "banana$".

  Key property: First column F = sorted(BWT)
    L: a n n b $ a a
    F: $ a a a b n n

  LF-mapping: The i-th occurrence of character c in L
  corresponds to the i-th occurrence of c in F.

  Step-by-step reconstruction:
  ┌────┬───┬───┬─────────────────────────────┐
  │ i  │ F │ L │ LF-mapping                  │
  ├────┼───┼───┼─────────────────────────────┤
  │ 0  │ $ │ a₁│ a₁ in L → a₁ in F (row 1)  │
  │ 1  │ a₁│ n₁│ n₁ in L → n₁ in F (row 5)  │
  │ 2  │ a₂│ n₂│ n₂ in L → n₂ in F (row 6)  │
  │ 3  │ a₃│ b₁│ b₁ in L → b₁ in F (row 4)  │
  │ 4  │ b₁│ $₁│ $₁ in L → $₁ in F (row 0)  │
  │ 5  │ n₁│ a₂│ a₂ in L → a₂ in F (row 2)  │
  │ 6  │ n₂│ a₃│ a₃ in L → a₃ in F (row 3)  │
  └────┴───┴───┴─────────────────────────────┘

  Start at row 0 ($):
  Row 0: F=$, L=a → follow LF: a₁ → row 1
  Row 1: F=a, L=n → follow LF: n₁ → row 5
  Row 5: F=n, L=a → follow LF: a₂ → row 2
  Row 2: F=a, L=n → follow LF: n₂ → row 6
  Row 6: F=n, L=a → follow LF: a₃ → row 3
  Row 3: F=a, L=b → follow LF: b₁ → row 4
  Row 4: F=b, L=$ → follow LF: $₁ → row 0 (done)

  Reading F column: $ a n a n a b → reverse → "banana$" ✓
```

---

## 4. Efficient BWT via Suffix Array

```
  BWT[i] = S[SA[i] - 1]   (character before each sorted suffix)

  S = "banana$"
  SA = [6, 5, 3, 1, 0, 4, 2]

  BWT[0] = S[SA[0]-1] = S[5] = 'a'
  BWT[1] = S[SA[1]-1] = S[4] = 'n'
  BWT[2] = S[SA[2]-1] = S[2] = 'n'
  BWT[3] = S[SA[3]-1] = S[0] = 'b'
  BWT[4] = S[SA[4]-1] = S[-1] = '$'  (wraps around: S[6]='$')
  BWT[5] = S[SA[5]-1] = S[3] = 'a'
  BWT[6] = S[SA[6]-1] = S[1] = 'a'

  BWT = "annb$aa" ✓

  ┌────────────────────────────────────────────────┐
  │  Build suffix array O(n log n) → BWT in O(n)  │
  │  Much faster than sorting all rotations O(n²)  │
  └────────────────────────────────────────────────┘
```

---

## 5. FM-Index: Pattern Matching with BWT

```
  The FM-index uses BWT + auxiliary data for O(m) pattern search.

  Key components:
  1. BWT string (L column)
  2. C[c] = number of characters < c in the string
  3. Occ(c, i) = number of occurrences of c in L[0..i]

  Search for pattern P (backward search):
    start = 0, end = n
    for i = |P|-1 down to 0:
        c = P[i]
        start = C[c] + Occ(c, start - 1)
        end = C[c] + Occ(c, end - 1)
    
    if start < end: pattern found (end - start occurrences)
    else: not found

  Example: Find "ana" in BWT of "banana$"
    Start with full range [0, 7)
    Process 'a' (last char):  [1, 4)  (rows starting with 'a')
    Process 'n':              [5, 7)  (rows starting with 'na')
    Process 'a':              [2, 4)  (rows starting with 'ana')
    2 occurrences found ✓
```

---

## 6. Implementation

```python
def bwt_transform(s):
    """Compute BWT of string s (must end with $)."""
    n = len(s)
    # Build suffix array
    sa = sorted(range(n), key=lambda i: s[i:])
    # BWT = character before each sorted suffix
    return ''.join(s[sa[i] - 1] for i in range(n))

def inverse_bwt(bwt):
    """Reconstruct original string from BWT."""
    n = len(bwt)
    # Build LF-mapping
    # Sort to get first column
    first = sorted(range(n), key=lambda i: bwt[i])
    
    # T-ranking approach
    table = sorted(range(n), key=lambda i: bwt[i])
    
    # Follow the chain from $ position
    result = []
    idx = bwt.index('$')
    for _ in range(n):
        idx = table[idx]
        result.append(bwt[idx])
    
    return ''.join(result)
```

---

## 7. Applications

```
  ┌────────────────────────────────────────────────────────────┐
  │  1. Data compression: bzip2 uses BWT + MTF + entropy      │
  │  2. Bioinformatics: BWA, Bowtie (DNA read alignment)      │
  │  3. FM-index: compressed full-text index                   │
  │  4. Pattern counting: O(m) with FM-index                   │
  │  5. Compressed suffix array: SA stored implicitly via BWT  │
  │                                                             │
  │  Why BWT is revolutionary for genomics:                    │
  │  ┌──────────────────────────────────────────────────────┐  │
  │  │  Human genome: 3 billion characters                  │  │
  │  │  FM-index compresses to a few GB                     │  │
  │  │  Pattern matching in O(m) time, fits in RAM!         │  │
  │  └──────────────────────────────────────────────────────┘  │
  └────────────────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Component | Details |
|-----------|---------|
| BWT output | Last column of sorted rotation matrix |
| Compression | Groups similar chars → better RLE/Huffman |
| Inverse BWT | LF-mapping in O(n) |
| Via suffix array | BWT[i] = S[SA[i] - 1] |
| FM-index search | O(m) pattern matching |
| Space (FM-index) | O(n) compressed |
| Used by | bzip2, BWA, Bowtie, BLAST+ |

---

## ❓ Quick Revision Questions

1. **What is the BWT of a string?**
   <details><summary>Answer</summary>The BWT is the last column of the matrix formed by sorting all rotations of the string lexicographically. It can be computed efficiently as BWT[i] = S[SA[i]-1] using the suffix array.</details>

2. **Why does BWT improve compressibility?**
   <details><summary>Answer</summary>BWT groups characters that share the same following context. For example, characters that frequently precede 'e' in English text ('h', 't', etc.) cluster together, creating runs that compress well with RLE.</details>

3. **What is the LF-mapping?**
   <details><summary>Answer</summary>The LF-mapping connects the last column (L) to the first column (F): the i-th occurrence of character c in L maps to the i-th occurrence of c in F. This enables inverse BWT reconstruction.</details>

4. **How does the FM-index search for patterns?**
   <details><summary>Answer</summary>Backward search: process pattern characters from right to left, narrowing the range of matching rows using C[c] (cumulative counts) and Occ(c,i) (occurrence counts in L). Range becomes empty → no match.</details>

5. **What is the time complexity of BWT construction via suffix array?**
   <details><summary>Answer</summary>O(n log n) for suffix array construction + O(n) for BWT extraction = O(n log n) total. With SA-IS or DC3 algorithm, the suffix array can be built in O(n), making overall BWT construction O(n).</details>

---

| [⬅️ Previous: Suffix Tree](02-suffix-tree.md) | [Next: Aho-Corasick ➡️](04-aho-corasick.md) |
|:---|---:|
