# Chapter 6.6 — Double Hashing: The Anti-Collision Shield

> **Unit 6: String Hashing** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Double hashing uses two independent hash functions to virtually eliminate
collision probability. This chapter covers implementation, when to use it,
and the math behind its reliability.

---

## 1. Why Double Hash?

```
  Single hash:  P(collision) ≈ 1/M ≈ 10⁻⁹
  
  After 10⁵ comparisons: P(any collision) ≈ 10⁻⁴ (0.01%)
  After 10⁶ comparisons: P(any collision) ≈ 10⁻³ (0.1%)
  After 10⁸ comparisons: P(any collision) ≈ 10%
  
  Double hash: P(collision) ≈ 1/(M₁×M₂) ≈ 10⁻¹⁸
  
  After 10⁹ comparisons: P(any collision) ≈ 10⁻⁹ (essentially 0)
  
  ┌──────────────────────────────────────────────┐
  │  Double hashing is practically PERFECT.      │
  │  You can treat hash equality as true         │
  │  equality without verification.              │
  └──────────────────────────────────────────────┘
```

---

## 2. Implementation

```python
class DoubleStringHash:
    """String hashing with two independent hash functions."""
    
    def __init__(self, s, p1=31, m1=10**9+7, p2=37, m2=10**9+9):
        n = len(s)
        self.m1, self.m2 = m1, m2
        
        # Hash arrays
        self.h1 = [0] * (n + 1)
        self.h2 = [0] * (n + 1)
        
        # Power arrays
        self.pw1 = [1] * (n + 1)
        self.pw2 = [1] * (n + 1)
        
        for i in range(n):
            c = ord(s[i]) - ord('a') + 1
            self.h1[i+1] = (self.h1[i] + c * self.pw1[i]) % m1
            self.h2[i+1] = (self.h2[i] + c * self.pw2[i]) % m2
            self.pw1[i+1] = (self.pw1[i] * p1) % m1
            self.pw2[i+1] = (self.pw2[i] * p2) % m2
    
    def get_hash(self, l, r):
        """Get double hash of s[l..r] as a tuple."""
        raw1 = (self.h1[r+1] - self.h1[l] + self.m1) % self.m1
        raw2 = (self.h2[r+1] - self.h2[l] + self.m2) % self.m2
        return (raw1, raw2)
    
    def equal(self, l1, r1, l2, r2):
        """Check if s[l1..r1] == s[l2..r2]."""
        if r1 - l1 != r2 - l2:
            return False
        h1a = (self.h1[r1+1] - self.h1[l1] + self.m1) % self.m1
        h1b = (self.h1[r2+1] - self.h1[l2] + self.m1) % self.m1
        h2a = (self.h2[r1+1] - self.h2[l1] + self.m2) % self.m2
        h2b = (self.h2[r2+1] - self.h2[l2] + self.m2) % self.m2
        
        return (h1a * self.pw1[l2] % self.m1 == h1b * self.pw1[l1] % self.m1 and
                h2a * self.pw2[l2] % self.m2 == h2b * self.pw2[l1] % self.m2)
```

---

## 3. Using Hash Tuples

```
  Key technique: Use (hash1, hash2) as a single identifier.
  Python tuples can be put in sets and used as dictionary keys.

  This gives you a combined "virtual modulus" of M₁ × M₂ ≈ 10¹⁸.

  Example: Count distinct substrings of length k
  ──────────────────────────────────────────────
  seen = set()
  for i in range(n - k + 1):
      h = dh.get_hash(i, i + k - 1)   # returns (h1, h2) tuple
      seen.add(h)
  return len(seen)
```

---

## 4. Performance Comparison

```
  ┌──────────────────────────────────────────────────────────┐
  │  Metric          │ Single Hash     │ Double Hash        │
  ├──────────────────┼─────────────────┼────────────────────┤
  │  Storage per hash│ 1 integer       │ 2 integers (tuple) │
  │  Comparison      │ 1 mod multiply  │ 2 mod multiplies   │
  │  Memory          │ ×1              │ ×2                 │
  │  Speed           │ ×1              │ ×1.5 to ×2         │
  │  Safety          │ 10⁻⁹ per comp  │ 10⁻¹⁸ per comp    │
  │  Safe comparisons│ ~10⁴            │ ~10⁹              │
  └──────────────────┴─────────────────┴────────────────────┘

  Double hashing costs ~2× more computation but is
  astronomically more reliable. Always use it when:
  - The problem could have anti-hash tests
  - You're doing > 10⁵ hash comparisons
```

---

## 5. Common Parameter Choices

```
  ┌────────────────────────────────────────────────┐
  │  Choice 1 (most common):                      │
  │    p₁ = 31,  M₁ = 10⁹ + 7                   │
  │    p₂ = 37,  M₂ = 10⁹ + 9                   │
  │                                                │
  │  Choice 2 (larger bases):                     │
  │    p₁ = 131, M₁ = 10⁹ + 7                   │
  │    p₂ = 137, M₂ = 10⁹ + 9                   │
  │                                                │
  │  Choice 3 (Mersenne prime):                   │
  │    p₁ = 31,  M₁ = 2³¹ - 1 = 2147483647      │
  │    p₂ = 37,  M₂ = 2⁶¹ - 1 (if 64-bit avail) │
  │                                                │
  │  Choice 4 (anti-hack):                        │
  │    p₁ = random prime                          │
  │    p₂ = different random prime                │
  │    M₁, M₂ = two large primes                 │
  └────────────────────────────────────────────────┘

  The key requirement: the two functions must be INDEPENDENT.
  Different p AND different M ensure independence.
```

---

## 6. Template for Competitive Programming

```python
import sys
from random import randint

def solve():
    # Anti-hack: randomize parameters
    P1, M1 = 31, 10**9 + 7
    P2, M2 = 37, 10**9 + 9
    
    s = input()
    n = len(s)
    
    # Build prefix hashes with both functions
    h1 = [0] * (n + 1)
    h2 = [0] * (n + 1)
    pw1 = [1] * (n + 1)
    pw2 = [1] * (n + 1)
    
    for i in range(n):
        c = ord(s[i]) - ord('a') + 1
        h1[i+1] = (h1[i] + c * pw1[i]) % M1
        h2[i+1] = (h2[i] + c * pw2[i]) % M2
        pw1[i+1] = pw1[i] * P1 % M1
        pw2[i+1] = pw2[i] * P2 % M2
    
    def get_hash(l, r):
        """O(1) double hash of s[l..r]."""
        return ((h1[r+1] - h1[l]) % M1,
                (h2[r+1] - h2[l]) % M2)
    
    # Now use get_hash() for your solution...
    # Hash tuples can go directly into sets/dicts.
```

---

## 📝 Summary Table

| Property | Single Hash | Double Hash |
|----------|-------------|-------------|
| Collision probability | 1/M ≈ 10⁻⁹ | 1/(M₁M₂) ≈ 10⁻¹⁸ |
| Birthday threshold | ~45K strings | ~1.4B strings |
| Memory overhead | 1× | 2× |
| Speed overhead | 1× | ~1.5-2× |
| Anti-hack safe | No | Mostly |
| Need verification | Sometimes | Almost never |

---

## ❓ Quick Revision Questions

1. **What is the birthday paradox threshold for double hashing?**
   <details><summary>Answer</summary>√(2 × M₁ × M₂) ≈ √(2 × 10¹⁸) ≈ 1.4 × 10⁹. About 1.4 billion strings needed for 50% collision probability.</details>

2. **Why must the two hash functions be independent?**
   <details><summary>Answer</summary>If they're correlated, collisions in one function would predict collisions in the other, reducing the combined safety. Independence ensures the probabilities truly multiply.</details>

3. **How do you store a double hash in a data structure?**
   <details><summary>Answer</summary>As a tuple (h1, h2). In Python, tuples are hashable and can be used in sets and as dictionary keys directly.</details>

4. **Is double hashing always necessary?**
   <details><summary>Answer</summary>No. For small-scale problems with few comparisons (< 10⁴) and non-adversarial inputs, single hashing with a large prime is sufficient.</details>

5. **What makes two hash functions "independent"?**
   <details><summary>Answer</summary>Different base p AND different modulus M. Using the same M with different p, or vice versa, still provides good independence in practice.</details>

---

| [⬅️ Previous: Longest Duplicate Substring](05-longest-duplicate-substring.md) | [Next Unit: Trie Applications ➡️](../07-Trie-Applications/01-trie-for-pattern-matching.md) |
|:---|---:|
