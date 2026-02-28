# 8.4 Comparing Substrings

## Unit 8: String Hashing Problems

---

## The Problem

> Compare any two substrings of a string in $O(1)$ time after $O(n)$ preprocessing.

```
╔══════════════════════════════════════════════════════════════╗
║  s = "abcabcabc"                                            ║
║                                                              ║
║  Is s[0..2] == s[3..5]?   "abc" == "abc" → YES             ║
║  Is s[0..2] == s[6..8]?   "abc" == "abc" → YES             ║
║  Is s[0..3] == s[3..6]?   "abca" == "abca" → YES           ║
║  Is s[1..3] == s[4..6]?   "bca" == "bca" → YES             ║
║                                                              ║
║  Naive: O(length) per comparison                            ║
║  Hash:  O(1) per comparison, O(n) preprocessing             ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Approach: Prefix Hash Array

Precompute prefix hashes so that any substring hash can be extracted in $O(1)$:

```
FUNCTION preprocess(s, d, q):
    n = len(s)
    prefHash = new array[n + 1]
    pw = new array[n + 1]

    prefHash[0] = 0
    pw[0] = 1

    FOR i = 1 TO n:
        prefHash[i] = (prefHash[i-1] * d + s[i-1]) % q
        pw[i] = (pw[i-1] * d) % q

    RETURN (prefHash, pw)

FUNCTION getHash(prefHash, pw, l, r, q):
    // Hash of s[l..r] (0-indexed, inclusive)
    len = r - l + 1
    h = (prefHash[r + 1] - prefHash[l] * pw[len]) % q
    IF h < 0: h += q
    RETURN h

FUNCTION compareSubstrings(prefHash, pw, l1, r1, l2, r2, q):
    RETURN getHash(prefHash, pw, l1, r1, q) ==
           getHash(prefHash, pw, l2, r2, q)
```

---

## Mathematical Derivation

```
╔══════════════════════════════════════════════════════════════╗
║  prefHash[i] = hash of s[0..i-1]                           ║
║             = s[0]·d^(i-1) + s[1]·d^(i-2) + ... + s[i-1]  ║
║                                                              ║
║  We want hash of s[l..r]:                                   ║
║  = s[l]·d^(r-l) + s[l+1]·d^(r-l-1) + ... + s[r]           ║
║                                                              ║
║  Notice:                                                    ║
║  prefHash[r+1] = s[0]·d^r + ... + s[l-1]·d^(r-l+1)        ║
║                + s[l]·d^(r-l) + ... + s[r]                  ║
║                  ↑────────────────────────↑                 ║
║                  this is what we want                       ║
║                                                              ║
║  prefHash[l] * d^(r-l+1) = s[0]·d^r + ... + s[l-1]·d^(r-l+1)║
║                                                              ║
║  Subtract: prefHash[r+1] - prefHash[l] * d^(r-l+1)         ║
║          = s[l]·d^(r-l) + ... + s[r]                        ║
║          = hash(s[l..r])  ✓                                 ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Complete Trace

```
s = "abcabc", d = 31, q = 997
(Using a→1, b→2, c→3)

Precompute prefix hashes:
  pH[0] = 0
  pH[1] = (0*31 + 1) % 997 = 1              (a)
  pH[2] = (1*31 + 2) % 997 = 33             (ab)
  pH[3] = (33*31 + 3) % 997 = 1026 % 997 = 29   (abc)
  pH[4] = (29*31 + 1) % 997 = 900           (abca)
  pH[5] = (900*31 + 2) % 997 = 27902 % 997 = 935  (abcab)
  pH[6] = (935*31 + 3) % 997 = 28988 % 997 = 63   (abcabc)

Precompute powers:
  pw[0] = 1
  pw[1] = 31
  pw[2] = 961
  pw[3] = 31*961 % 997 = 29791 % 997 = 889
  pw[4] = 31*889 % 997 = 27559 % 997 = 638
  pw[5] = 31*638 % 997 = 19778 % 997 = 838
  pw[6] = 31*838 % 997 = 25978 % 997 = 45

Query 1: Is s[0..2] == s[3..5]?  ("abc" vs "abc")
  hash(0,2) = (pH[3] - pH[0]*pw[3]) % 997
            = (29 - 0) % 997 = 29
  hash(3,5) = (pH[6] - pH[3]*pw[3]) % 997
            = (63 - 29*889) % 997
            = (63 - 25781) % 997
            = -25718 % 997
            = -25718 + 26*997 = -25718 + 25922 = ... 
            Let me recalculate: -25718 mod 997
            -25718 / 997 ≈ -25.8 → floor = -26
            -25718 - (-26)*997 = -25718 + 25922 = 204
            
Hmm, let me re-verify pH[3]:
  pH[3] = (33*31 + 3) = 1023 + 3 = 1026 % 997 = 29 ✓

OK, let me use simpler numbers. d=10, q=97:

s = "abcabc" → values [1, 2, 3, 1, 2, 3]

pH[0] = 0
pH[1] = (0*10 + 1) % 97 = 1
pH[2] = (1*10 + 2) % 97 = 12
pH[3] = (12*10 + 3) % 97 = 123 % 97 = 26
pH[4] = (26*10 + 1) % 97 = 261 % 97 = 67
pH[5] = (67*10 + 2) % 97 = 672 % 97 = 90
pH[6] = (90*10 + 3) % 97 = 903 % 97 = 27

pw[0] = 1, pw[1] = 10, pw[2] = 100%97 = 3
pw[3] = 30, pw[4] = 300%97 = 9, pw[5] = 90

hash(0,2) = (pH[3] - pH[0]*pw[3]) % 97 = (26 - 0) % 97 = 26
hash(3,5) = (pH[6] - pH[3]*pw[3]) % 97
          = (27 - 26*30) % 97
          = (27 - 780) % 97
          = -753 % 97
          = -753 + 8*97 = -753 + 776 = 23

Hmm, these don't match because my direct hash of "123" should be:
  1*100 + 2*10 + 3 = 123, 123 % 97 = 26 ✓

And "123" starting at position 3:
  Same sequence → direct hash = 1*100 + 2*10 + 3 = 123, 123%97 = 26

Let me recheck:
pH[6] = hash("abcabc") = 123123 % 97
  123123 / 97 = 1269.3... → 1269*97 = 123093
  123123 - 123093 = 30
  
So pH[6] should be 30, not 27. Let me redo:
pH[4] = (26*10 + 1) % 97 = 261 % 97 = 261 - 2*97 = 67 ✓
pH[5] = (67*10 + 2) % 97 = 672 % 97 = 672 - 6*97 = 672-582 = 90 ✓
pH[6] = (90*10 + 3) % 97 = 903 % 97 = 903 - 9*97 = 903-873 = 30 ✓

Redo: hash(3,5) = (pH[6] - pH[3]*pw[3]) % 97
     = (30 - 26*30) % 97
     = (30 - 780) % 97
     = -750 % 97
     = -750 + 8*97 = -750 + 776 = 26 ✓

hash(0,2) = 26, hash(3,5) = 26 → MATCH! ✓
"abc" == "abc" confirmed by hash comparison in O(1)
```

---

## Applications

### 1. Longest Common Prefix of Two Suffixes

```
FUNCTION longestCommonPrefix(s, i, j):
    // Binary search on the length of common prefix
    lo = 0, hi = min(len(s) - i, len(s) - j)

    WHILE lo < hi:
        mid = (lo + hi + 1) / 2
        IF getHash(i, i+mid-1) == getHash(j, j+mid-1):
            lo = mid
        ELSE:
            hi = mid - 1

    RETURN lo

// Time: O(log n) per query after O(n) preprocessing
```

### 2. Lexicographic Comparison in O(log n)

```
FUNCTION compareLex(s, l1, r1, l2, r2):
    // Find first differing position using binary search + hash
    lcp = longestCommonPrefix(s, l1, l2)
    len1 = r1 - l1 + 1
    len2 = r2 - l2 + 1

    IF lcp >= min(len1, len2):
        RETURN len1 - len2   // shorter is "smaller"

    RETURN s[l1 + lcp] - s[l2 + lcp]   // compare first diff char
```

### 3. Count Distinct Substrings

```
FUNCTION countDistinct(s):
    n = len(s)
    hashes = new HashSet()

    FOR length = 1 TO n:
        FOR start = 0 TO n - length:
            h = getHash(start, start + length - 1)
            hashes.add(h)

    RETURN hashes.size()

// Time: O(n²), Space: O(n²)
```

---

## Quick Revision Question

**Q: After preprocessing a string of length $n$, how much time does it take to determine if two substrings of length $k$ are equal? What is the probability of a false positive?**

<details>
<summary>Click to reveal answer</summary>

**Time: $O(1)$** — compute each substring hash using:
$$\text{hash}(l, r) = (\text{prefHash}[r+1] - \text{prefHash}[l] \times d^{r-l+1}) \mod q$$

This uses only precomputed arrays (prefix hashes and powers), so it's constant-time arithmetic.

**False positive probability:** $\leq \frac{k-1}{q}$ (by Schwartz-Zippel lemma)

For $q = 10^9 + 7$ and $k = 1000$:
$$\Pr[\text{false positive}] \leq \frac{999}{10^9 + 7} \approx 10^{-6}$$

To reduce this further, use **double hashing** (two independent hash functions). The combined false positive probability becomes:
$$\Pr \leq \frac{k-1}{q_1} \times \frac{k-1}{q_2} \approx 10^{-12}$$

In practice, for competitive programming and most applications, a single hash with $q \approx 10^9$ is sufficient. For production systems requiring guaranteed correctness, verify with character comparison when hashes match.

</details>

---

| [← Polynomial Hashing](03-polynomial-hashing.md) | [Next: String Matching Applications →](05-string-matching-applications.md) |
|:---|---:|
