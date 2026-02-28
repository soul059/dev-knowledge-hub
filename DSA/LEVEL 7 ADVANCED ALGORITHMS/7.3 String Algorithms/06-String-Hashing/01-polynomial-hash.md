# Chapter 6.1 — Polynomial Hashing for Strings

> **Unit 6: String Hashing** | [Course Home](../README.md)

---

## 📋 Chapter Overview

String hashing converts a string into a number, enabling O(1) string comparisons
(with high probability). Polynomial hashing is the most widely used technique,
forming the basis for Rabin-Karp, substring comparison, and many CP problems.

---

## 1. The Core Formula

```
  hash(S) = (S[0]×p⁰ + S[1]×p¹ + S[2]×p² + ... + S[n-1]×pⁿ⁻¹) mod M

  Where:
    p = base (a prime, e.g., 31 or 37)
    M = modulus (a large prime, e.g., 10⁹+7 or 10⁹+9)
    S[i] = character value (e.g., 'a'=1, 'b'=2, ..., 'z'=26)

  ⚠️ Use 1-based character mapping! ('a'=1, not 'a'=0)
     If 'a'=0, then "a", "aa", "aaa" all hash to 0. Bad!
```

---

## 2. Computing Prefix Hashes

```
  Instead of hashing each substring independently (expensive),
  precompute PREFIX HASHES for O(1) substring hash queries.

  h[0] = 0
  h[i] = h[i-1] + S[i-1] × p^(i-1)   (mod M)

  h[i] stores the hash of S[0..i-1].

  Example: S = "abcab", p = 31, M = 10⁹+7
  
  h[0] = 0
  h[1] = 0 + 1×31⁰   = 1         (a=1)
  h[2] = 1 + 2×31¹    = 63        (b=2)
  h[3] = 63 + 3×31²   = 2946      (c=3)  (31²=961, 3×961=2883)
         Wait: 63 + 2883 = 2946 ✓
  h[4] = 2946 + 1×31³ = 2946 + 29791 = 32737     (a=1)
  h[5] = 32737 + 2×31⁴ = 32737 + 1845098 = 1877835  (b=2)
```

---

## 3. Substring Hash Query

```
  To get hash of S[l..r] (0-indexed):

  hash(S[l..r]) = (h[r+1] - h[l]) × p^(-l)  mod M

  Or normalize by comparing:
  hash(S[l..r]) × p^l = h[r+1] - h[l]  mod M

  For COMPARISON of two substrings S[l₁..r₁] and S[l₂..r₂]:
  (h[r₁+1] - h[l₁]) × p^l₂ == (h[r₂+1] - h[l₂]) × p^l₁  mod M

  ┌──────────────────────────────────────────────────────┐
  │  Simpler approach: Store and compare directly.       │
  │  hash(l, r) = (h[r+1] - h[l]) × inv_p[l]  mod M   │
  │  where inv_p[l] = modular inverse of p^l            │
  └──────────────────────────────────────────────────────┘
```

---

## 4. Precomputation Code

```python
class StringHash:
    def __init__(self, s, p=31, mod=10**9 + 7):
        self.mod = mod
        self.n = len(s)
        
        # Prefix hashes
        self.h = [0] * (self.n + 1)
        # Powers of p
        self.pw = [1] * (self.n + 1)
        
        for i in range(self.n):
            self.h[i + 1] = (self.h[i] + (ord(s[i]) - ord('a') + 1) * self.pw[i]) % mod
            self.pw[i + 1] = (self.pw[i] * p) % mod
    
    def get_hash(self, l, r):
        """Get hash of s[l..r] (0-indexed, inclusive)."""
        # Returns (h[r+1] - h[l]) mod M, scaled by p^l
        return (self.h[r + 1] - self.h[l] + self.mod) % self.mod
    
    def compare(self, l1, r1, l2, r2):
        """Check if s[l1..r1] == s[l2..r2] using hashing."""
        if r1 - l1 != r2 - l2:
            return False
        h1 = self.get_hash(l1, r1)
        h2 = self.get_hash(l2, r2)
        # Normalize by multiplying by appropriate power
        return (h1 * self.pw[l2]) % self.mod == (h2 * self.pw[l1]) % self.mod


# Example
sh = StringHash("abcabc")
print(sh.compare(0, 2, 3, 5))  # "abc" == "abc" → True
print(sh.compare(0, 2, 1, 3))  # "abc" == "bca" → False
```

---

## 5. Why Characters Start at 1

```
  If 'a' = 0:
    hash("a")   = 0×31⁰ = 0
    hash("aa")  = 0×31⁰ + 0×31¹ = 0
    hash("aaa") = 0

  All strings of only 'a' hash to 0!
  Massive collision problem.

  If 'a' = 1:
    hash("a")   = 1
    hash("aa")  = 1 + 1×31 = 32
    hash("aaa") = 1 + 31 + 961 = 993

  All different! ✓
```

---

## 6. Choosing p and M

```
  ┌──────────────────────────────────────────────────┐
  │  For lowercase letters only:                     │
  │  p = 31 (smallest prime > 26)                   │
  │  M = 10⁹+7 or 10⁹+9                            │
  │                                                  │
  │  For mixed case + digits:                       │
  │  p = 53 (prime > 52 letters + digits)           │
  │  M = 10⁹+7                                      │
  │                                                  │
  │  For full ASCII:                                 │
  │  p = 131 or 137                                 │
  │  M = 10⁹+7                                      │
  └──────────────────────────────────────────────────┘

  Rules:
  1. p should be coprime to M (both prime → automatic)
  2. p > alphabet size
  3. M should be large prime
  4. p × M should fit in 64-bit integer
```

---

## 7. Hash Collision Probability

```
  For two random strings of the same length:
  P(collision) ≈ 1/M

  With M = 10⁹+7:
  ┌───────────────────────────────────────────────────────┐
  │  1 comparison:     P ≈ 10⁻⁹ (1 in a billion)       │
  │  10⁶ comparisons:  P ≈ 10⁻³ (1 in a thousand)      │
  │  10⁸ comparisons:  P ≈ 10⁻¹ (10% chance of error‼) │
  └───────────────────────────────────────────────────────┘

  For competitive programming with many comparisons,
  10⁹+7 alone may not be enough. Use double hashing!
```

---

## 📝 Summary Table

| Component | Details |
|-----------|---------|
| Formula | hash = Σ S[i]×p^i mod M |
| Base p | 31 (lowercase), 53 (mixed) |
| Modulus M | 10⁹+7 (large prime) |
| Character mapping | 'a'=1, NOT 'a'=0 |
| Prefix hash | h[i] = hash of S[0..i-1] |
| Substring hash | O(1) using prefix hashes |
| Precomputation | O(n) time and space |
| Collision probability | ≈ 1/M per comparison |

---

## ❓ Quick Revision Questions

1. **Why must we map 'a' to 1 instead of 0?**
   <details><summary>Answer</summary>If 'a'=0, strings like "a", "aa", "aaa" all hash to 0, causing many collisions. Starting from 1 ensures every character contributes a nonzero value.</details>

2. **What is stored in h[i]?**
   <details><summary>Answer</summary>The hash of the prefix S[0..i-1], computed as S[0]×p⁰ + S[1]×p¹ + ... + S[i-1]×p^(i-1) mod M.</details>

3. **How do you compare two substrings in O(1)?**
   <details><summary>Answer</summary>Compute their hashes using prefix hashes: (h[r+1]-h[l]) for each, then normalize by multiplying by appropriate powers of p to align the polynomial terms.</details>

4. **What is the collision probability for a single comparison?**
   <details><summary>Answer</summary>Approximately 1/M. With M = 10⁹+7, that's about 10⁻⁹ per comparison.</details>

5. **Why should p be larger than the alphabet size?**
   <details><summary>Answer</summary>If p ≤ alphabet size, different characters can produce the same value modulo p, increasing collision probability. A prime p > |Σ| ensures better distribution.</details>

---

| [⬅️ Previous Unit: Z Algorithm Use Cases](../05-Z-Algorithm/05-use-cases.md) | [Next: Rolling Hash ➡️](02-rolling-hash.md) |
|:---|---:|
