# Chapter 6.3 — Hash Collisions & Mitigation

> **Unit 6: String Hashing** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Hash collisions — when different strings produce the same hash — are unavoidable.
This chapter analyzes collision probability, demonstrates anti-hash attacks, and
presents mitigation strategies including double hashing.

---

## 1. Why Collisions Are Inevitable

```
  Pigeonhole Principle:
  ─────────────────────
  Number of strings of length m over alphabet Σ: |Σ|^m
  Number of hash slots: M

  For m = 10, Σ = 26: 26¹⁰ ≈ 1.4 × 10¹⁴ strings
  For M = 10⁹+7:      10⁹ slots

  Average strings per slot: 1.4 × 10⁵ = 140,000!

  Collisions are mathematically guaranteed.
  The question is: HOW LIKELY are they in practice?
```

---

## 2. Birthday Paradox & Hash Collisions

```
  If you hash n different strings, the probability of
  AT LEAST ONE collision follows the birthday paradox:

  P(collision) ≈ 1 - e^(-n²/(2M))

  ┌─────────────────────────────────────────────────┐
  │  M = 10⁹+7                                     │
  │                                                  │
  │  n = 1000:     P ≈ 0.05%   (very safe)         │
  │  n = 10,000:   P ≈ 5%      (starting to worry) │
  │  n = 50,000:   P ≈ 71%     (likely collision!)  │
  │  n = 100,000:  P ≈ 99.3%   (almost certain)    │
  │                                                  │
  │  50% collision threshold: n ≈ √(2M) ≈ 44,721  │
  └─────────────────────────────────────────────────┘

  With only ~45K hashed strings, collision is 50% likely!
  This is much less than you might expect.
```

---

## 3. Anti-Hash Tests (Adversarial Inputs)

```
  In competitive programming, problem setters can create
  inputs that force collisions for a specific hash function.

  How to find collisions for hash(p=31, M=10⁹+7):
  ─────────────────────────────────────────────────
  Generate many strings, find two with the same hash.
  By birthday paradox, only ~45K strings needed.

  The attacker knows your p and M → can craft inputs that:
  1. Make all strings hash to the same value
  2. Create O(n²) comparisons
  3. Turn O(n) solution into O(n²) TLE

  This is why fixed hash parameters are DANGEROUS!
```

---

## 4. Mitigation Strategy 1: Double Hashing

```
  Use TWO independent hash functions:

  Hash 1: p₁ = 31,  M₁ = 10⁹ + 7
  Hash 2: p₂ = 37,  M₂ = 10⁹ + 9

  Two strings match only if BOTH hashes agree.

  P(collision) = 1/M₁ × 1/M₂ ≈ 10⁻¹⁸

  Birthday paradox threshold: √(2 × M₁ × M₂) ≈ 1.4 × 10⁹

  You need ~1.4 BILLION strings for a 50% collision chance!
  Virtually collision-free.
```

```python
class DoubleHash:
    def __init__(self, s):
        self.n = len(s)
        p1, m1 = 31, 10**9 + 7
        p2, m2 = 37, 10**9 + 9
        
        self.h1 = [0] * (self.n + 1)
        self.h2 = [0] * (self.n + 1)
        self.pw1 = [1] * (self.n + 1)
        self.pw2 = [1] * (self.n + 1)
        
        for i in range(self.n):
            c = ord(s[i]) - ord('a') + 1
            self.h1[i+1] = (self.h1[i] + c * self.pw1[i]) % m1
            self.h2[i+1] = (self.h2[i] + c * self.pw2[i]) % m2
            self.pw1[i+1] = (self.pw1[i] * p1) % m1
            self.pw2[i+1] = (self.pw2[i] * p2) % m2
        
        self.m1, self.m2 = m1, m2
    
    def get_hash(self, l, r):
        """Return (hash1, hash2) tuple for s[l..r]."""
        raw1 = (self.h1[r+1] - self.h1[l] + self.m1) % self.m1
        raw2 = (self.h2[r+1] - self.h2[l] + self.m2) % self.m2
        return (raw1, raw2)
    
    def compare(self, l1, r1, l2, r2):
        """Compare s[l1..r1] with s[l2..r2]."""
        if r1-l1 != r2-l2:
            return False
        h1a, h2a = self.get_hash(l1, r1)
        h1b, h2b = self.get_hash(l2, r2)
        return (h1a * self.pw1[l2] % self.m1 == h1b * self.pw1[l1] % self.m1 and
                h2a * self.pw2[l2] % self.m2 == h2b * self.pw2[l1] % self.m2)
```

---

## 5. Mitigation Strategy 2: Randomized Base

```
  Choose p randomly at runtime:
  
  import random
  p = random.randint(26, 10**9)

  The attacker doesn't know which p you'll use,
  so they can't craft adversarial inputs.

  Combined with a fixed large M, this is usually sufficient.
```

---

## 6. Mitigation Strategy 3: Multiple Moduli

```
  Instead of one modulus, use pair of moduli:

  Use the hash PAIR as the identifier:
  (hash mod M₁, hash mod M₂)

  Equivalent to using modulus M₁ × M₂ ≈ 10¹⁸.

  This is essentially how double hashing works,
  but can be done with a single base.
```

---

## 7. When Single Hashing Is Enough

```
  ┌──────────────────────────────────────────────────┐
  │  Single hash is OK when:                         │
  │  • Few comparisons (< 10⁴)                      │
  │  • Non-adversarial inputs (random data)          │
  │  • Quick prototype or proof-of-concept           │
  │                                                   │
  │  Double hash is needed when:                     │
  │  • Many comparisons (> 10⁵)                      │
  │  • Adversarial inputs possible (competitive)     │
  │  • Correctness is critical (no WA tolerance)     │
  │  • Problem has anti-hash tests                   │
  └──────────────────────────────────────────────────┘
```

---

## 8. Collision Detection & Verification

```
  Even with hashing, you can verify critical results:

  Strategy: "Hash first, verify if needed"
  
  1. Use hash to quickly filter candidates
  2. For the final answer, verify with actual comparison
  
  Example: Finding longest common substring
  - Binary search on length L
  - Hash all substrings of length L from both strings
  - If hash sets intersect → candidate found
  - Verify the candidate with actual string comparison
  - Total: O(n log n) with hash + O(L) for final verification
```

---

## 📝 Summary Table

| Strategy | Collision Probability | Extra Cost | When to Use |
|----------|----------------------|------------|-------------|
| Single hash (M ≈ 10⁹) | 10⁻⁹ per comparison | ×1 | Non-adversarial, few comparisons |
| Double hash | 10⁻¹⁸ per comparison | ×2 | Competitive programming |
| Random base | Depends on M | ×1 | Anti-hack protection |
| Hash + verify | 0 (deterministic) | +O(m) verify | Must be 100% correct |

---

## ❓ Quick Revision Questions

1. **At what n does the birthday paradox predict 50% collision chance with M = 10⁹+7?**
   <details><summary>Answer</summary>n ≈ √(2M) ≈ √(2 × 10⁹) ≈ 44,721. With only ~45K strings, there's a 50% chance of at least one collision.</details>

2. **How does double hashing reduce collision probability?**
   <details><summary>Answer</summary>Two independent hash functions must both collide simultaneously. The probability is 1/M₁ × 1/M₂ ≈ 10⁻¹⁸, compared to 1/M ≈ 10⁻⁹ for single hashing.</details>

3. **What is an anti-hash test?**
   <details><summary>Answer</summary>An adversarially crafted input designed to create maximum hash collisions for a specific hash function, typically causing O(n²) behavior where O(n) is expected.</details>

4. **Why does randomizing the base help against adversarial inputs?**
   <details><summary>Answer</summary>The attacker must know the base p to craft colliding strings. A random base chosen at runtime is unpredictable, making targeted attacks impossible.</details>

5. **When would you use "hash first, verify later"?**
   <details><summary>Answer</summary>When the final answer must be deterministically correct but most candidates can be filtered quickly by hashing. The hash eliminates most non-matches in O(1), and you only do expensive O(m) verification on the few hash matches.</details>

---

| [⬅️ Previous: Rolling Hash](02-rolling-hash.md) | [Next: Substring Comparison ➡️](04-substring-comparison.md) |
|:---|---:|
