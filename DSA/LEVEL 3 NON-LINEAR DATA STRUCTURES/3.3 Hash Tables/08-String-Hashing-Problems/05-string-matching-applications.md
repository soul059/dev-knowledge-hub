# 8.5 String Matching Applications

## Unit 8: String Hashing Problems

---

## Overview

String hashing enables efficient solutions to a family of pattern matching problems beyond basic single-pattern search.

```
╔══════════════════════════════════════════════════════════════╗
║  APPLICATION              │ TECHNIQUE                       ║
║  ─────────────────────────┼─────────────────────────────────║
║  Single pattern search    │ Rabin-Karp / Rolling hash       ║
║  Multiple pattern search  │ Hash all patterns, scan text    ║
║  2D pattern search        │ Row hashes + column rolling     ║
║  Plagiarism detection     │ Document fingerprinting         ║
║  File deduplication       │ Content-defined chunking        ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 1. Multi-Pattern Matching

> Given text $T$ and $k$ patterns $P_1, P_2, \ldots, P_k$, find all occurrences.

### Naive: Run Rabin-Karp $k$ times → $O(kn)$

### Better: Hash all patterns into a set → $O(n + km)$

```
FUNCTION multiPatternSearch(text, patterns):
    // Group patterns by length
    byLength = new HashMap()    // length → set of hashes
    FOR each pattern in patterns:
        m = len(pattern)
        h = polyHash(pattern)
        byLength[m].add((h, pattern))

    results = new HashMap()    // pattern → [positions]

    // For each distinct length, do one pass over text
    FOR each (m, hashSet) in byLength:
        rh = computeHash(text[0..m-1])

        FOR i = 0 TO len(text) - m:
            IF i > 0:
                rh = rollHash(rh, text[i-1], text[i+m-1])

            FOR each (h, pat) in hashSet:
                IF rh == h AND text[i..i+m-1] == pat:
                    results[pat].add(i)

    RETURN results
```

### Trace

```
Text = "ABCABDABC"
Patterns = ["ABC", "ABD", "BC"]

Group by length:
  Length 3: {"ABC" → hash=X, "ABD" → hash=Y}
  Length 2: {"BC" → hash=Z}

Pass 1 (length=3): Scan text with window of 3
  i=0: "ABC" hash=X → matches "ABC" → found at 0
  i=1: "BCA" hash=? → no match
  i=2: "CAB" hash=? → no match
  i=3: "ABD" hash=Y → matches "ABD" → found at 3
  i=4: "BDA" hash=? → no match
  i=5: "DAB" hash=? → no match
  i=6: "ABC" hash=X → matches "ABC" → found at 6

Pass 2 (length=2): Scan text with window of 2
  i=0: "AB" → no match
  i=1: "BC" hash=Z → matches "BC" → found at 1
  i=7: "BC" hash=Z → matches "BC" → found at 7

Results: {"ABC": [0,6], "ABD": [3], "BC": [1,7]}
```

---

## 2. Two-Dimensional Pattern Matching

> Given an $N \times N$ text matrix and an $M \times M$ pattern matrix, find the pattern's position.

```
╔══════════════════════════════════════════════════════════════╗
║  Text (5×5):          Pattern (2×2):                        ║
║  A B C D E            B C                                   ║
║  F B C H I            B C                                   ║
║  J B C L M                                                  ║
║  N O P Q R            Found at position (1,1)               ║
║  S T U V W                                                  ║
╚══════════════════════════════════════════════════════════════╝
```

### Algorithm

```
FUNCTION search2D(text, pattern):
    N = rows(text), M = rows(pattern)

    // Step 1: Hash each row of pattern
    patternRowHashes = []
    FOR each row in pattern:
        patternRowHashes.add(rowHash(row))

    // Step 2: For each column position, hash M consecutive
    //         row-hashes vertically using rolling hash
    FOR col = 0 TO N - M:
        // Hash text rows at this column offset
        textRowHashes = []
        FOR row = 0 TO N - 1:
            textRowHashes.add(rowHash(text[row][col..col+M-1]))

        // Now do 1D Rabin-Karp on the column of row-hashes
        // matching against patternRowHashes
        IF contains(textRowHashes, patternRowHashes):
            RETURN (row, col)  // found!

    RETURN NOT_FOUND
```

**Time:** $O(N^2)$ average

---

## 3. Document Fingerprinting (Winnowing)

Used in plagiarism detection (e.g., MOSS, Turnitin):

```
╔══════════════════════════════════════════════════════════════╗
║  WINNOWING ALGORITHM:                                       ║
║                                                              ║
║  1. Compute hash for each k-gram (substring of length k)   ║
║  2. Slide a window of size w over the hash sequence         ║
║  3. Select the MINIMUM hash from each window position       ║
║  4. These selected hashes form the document "fingerprint"  ║
║                                                              ║
║  Hash sequence: [77, 74, 42, 17, 98, 12, 85, 63, ...]     ║
║  Window w=4:    [77, 74, 42, 17] → select 17              ║
║                     [74, 42, 17, 98] → select 17 (same)   ║
║                         [42, 17, 98, 12] → select 12      ║
║                             [17, 98, 12, 85] → select 12  ║
║                                                              ║
║  Fingerprint: {17, 12, ...}                                 ║
║                                                              ║
║  Compare fingerprints between documents:                    ║
║  High overlap → likely plagiarism                           ║
╚══════════════════════════════════════════════════════════════╝
```

### Similarity Measurement

```
similarity(doc1, doc2) = |fingerprint(doc1) ∩ fingerprint(doc2)|
                         ─────────────────────────────────────
                         |fingerprint(doc1) ∪ fingerprint(doc2)|

This is the Jaccard similarity coefficient.
```

---

## 4. Content-Defined Chunking (File Dedup)

```
╔══════════════════════════════════════════════════════════════╗
║  Fixed-size chunking:                                       ║
║  [chunk1|chunk2|chunk3|chunk4]                              ║
║  Insert 1 byte at start → ALL chunks shift → all different ║
║                                                              ║
║  Content-defined chunking (using rolling hash):             ║
║  Scan file, compute rolling hash at each byte              ║
║  When hash mod M == 0 → place chunk boundary here          ║
║                                                              ║
║  Boundaries depend on LOCAL content, not position           ║
║  → Insert/delete only affects nearby chunks                 ║
║  → Most chunks remain identical → better dedup!            ║
╚══════════════════════════════════════════════════════════════╝
```

Used in: rsync, git (packfiles), cloud storage deduplication.

---

## 5. Repeated Substring Check

> Check if there exists a substring of length $k$ that appears at least twice.

```
FUNCTION hasRepeatedSubstring(s, k):
    seen = new HashSet()

    FOR i = 0 TO len(s) - k:
        h = getHash(i, i + k - 1)   // O(1) with prefix hashes
        IF seen.contains(h):
            RETURN true
        seen.add(h)

    RETURN false

// Time: O(n), Space: O(n)
// Used as subroutine in binary search for longest repeated substring
```

---

## Comparison of String Matching Algorithms

| Algorithm | Preprocessing | Matching | Best For |
|-----------|:---:|:---:|---------|
| Naive | $O(1)$ | $O(nm)$ | Short patterns |
| Rabin-Karp | $O(m)$ | $O(n)$ avg | Multiple patterns |
| KMP | $O(m)$ | $O(n)$ | Single pattern, guaranteed |
| Boyer-Moore | $O(m+\sigma)$ | $O(n/m)$ avg | Long patterns |
| Aho-Corasick | $O(\sum m_i)$ | $O(n+z)$ | Many patterns |

```
$z$ = total number of matches, $\sigma$ = alphabet size
```

---

## Quick Revision Question

**Q: You need to find if any string in a list of 1000 strings (each of length ~100) appears as a substring in a large document of length 1,000,000. What approach would you use?**

<details>
<summary>Click to reveal answer</summary>

**Approach: Multi-pattern Rabin-Karp**

1. Compute the hash of each of the 1000 patterns → store in a HashSet
2. Slide a rolling hash window of length 100 (assuming all same length) across the document
3. At each position, check if the window hash exists in the HashSet
4. On match, verify characters to avoid false positives

**Time complexity:**
- Hash all patterns: $O(1000 \times 100) = O(100,000)$
- Scan document: $O(1,000,000)$ with $O(1)$ hash lookup per position
- Total: $O(1,100,000) \approx O(n)$
- Naive approach: $O(1000 \times 1,000,000 \times 100) = O(10^{11})$ — infeasible

**If patterns have different lengths**, group by length and do one pass per distinct length. With $L$ distinct lengths: $O(L \times n)$.

**Alternative:** For many patterns of varying lengths, **Aho-Corasick** is optimal at $O(n + \sum m_i + z)$ where $z$ is the number of matches. It builds a trie with failure links — essentially KMP generalized to multiple patterns.

</details>

---

| [← Comparing Substrings](04-comparing-substrings.md) | [Next: Longest Duplicate Substring →](06-longest-duplicate-substring.md) |
|:---|---:|
